# helprepo
void Animate(float dt) {
        if (state != started || spline.cps.size() < 2) return;

        // Apply gravity
        vec2 gravity = vec2(0, -g); // Assuming +y is upwards

        // Predict the next position based on current velocity and gravity
        vec2 nextPosition = position + velocity * dt + 0.5f * gravity * dt * dt;
        vec2 prevVelocity = velocity;
        velocity += gravity * dt;

        // Find the closest point on the spline to the predicted position
        float closestT = FindClosestT(nextPosition); // Implement this accurately

        vec3 splinePoint3D = spline.r(closestT);
        vec2 closestPointOnSpline = vec2(splinePoint3D.x, splinePoint3D.y);

        // Calculate the tangent of the spline at the closest point
        float epsilon = 0.001f;
        vec3 tangent3D = normalize(spline.r(closestT + epsilon) - spline.r(closestT - epsilon));
        vec2 tangent = normalize(vec2(tangent3D.x, tangent3D.y));
        vec2 normal = vec2(-tangent.y, tangent.x);

        // Vector from the closest spline point to the predicted position
        vec2 displacementFromTrack = nextPosition - closestPointOnSpline;
        float distanceToTrack = glm::length(displacementFromTrack);
        float trackProximityThreshold = 0.1f;

        float tangentialVelocity = 0.0f; // Declare and initialize here

        if (distanceToTrack > trackProximityThreshold) {
            // Gondola is moving away from the track (or has lost contact)
            position = nextPosition;
            if (distanceToTrack > 0.5f) {
                state = fallen;
                printf("Fallen!\n");
            }
        } else {
            // Gondola is close to the track, constrain motion

            // Project gravity onto the tangent to get tangential acceleration
            float tangentialGravity = glm::dot(gravity, tangent);
            velocity += tangentialGravity * dt;

            position = closestPointOnSpline;

            // Calculate tangential velocity after the update
            tangentialVelocity = glm::dot(velocity, tangent);

            // Check for "flight" (normal force direction)
            float normalForce = glm::dot(gravity, normal);
            if (normalForce > 0) { // Normal force pointing away from the track
                state = fallen;
                printf("Lost contact (normal force positive)!\n");
            }
        }

        // Update rotation based on the tangential velocity
        phi -= tangentialVelocity * dt / radius;
    }


    // Helper function to find the spline parameter 't' closest to a given point
    float FindClosestT(vec2 point, int numIterations = 50) {
        if (spline.cps.size() < 2) return 0.0f;
        float startT = spline.ts.front();
        float endT = spline.ts.back();
        float bestT = startT;
        float minDistSq = std::numeric_limits<float>::max();

        for (int i = 0; i < numIterations; ++i) {
           float t = startT + (endT - startT) * static_cast<float>(i) / (numIterations - 1);
           vec3 splinePoint3D = spline.r(t);
           vec2 splinePoint = vec2(splinePoint3D.x, splinePoint3D.y);
           float distSq = glm::distance(point, splinePoint);
           if (distSq < minDistSq) {
              minDistSq = distSq;
              bestT = t;
           }
        }
        return bestT;
    }

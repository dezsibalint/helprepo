![image](https://github.com/user-attachments/assets/ba38835c-210a-45dd-9410-5536c0d7996c)


```    

 class Gondola : public Geometry<vec2> {
public:
   enum State { waiting, started, fallen };
   State state = waiting;
   CatmullRom spline;
   Geometry<vec2>* track;
   vec2 position;
   vec2 velocity;
   float phi = 0;  // Rotation angle
   float radius = 0.1f;  // Wheel radius
   float mass = 1.0f;  // Gondola mass
   float g = 40.0f;  // Gravity

   Gondola(Geometry<vec2>* track) : track(track) {
      if (!track->Vtx().empty()) {
         for (vec2 pt : track->Vtx()) {
            spline.AddControlPoint(pt);
         }
         if (!spline.cps.empty()) {
            position = vec2(spline.cps.front().x, spline.cps.front().y);
         }
      } else {
         std::cerr << "Error: Track has no points!" << std::endl;
      }
   }

   void Start() {
      if (state == waiting) {
         printf("Started\n");
         position =  spline.r(0.01f);
         velocity = vec2(0, 0);
         phi = 0;
         state = started;
      }
   }

   void Animate(float dt) {
      if (state == waiting || spline.cps.size() < 2) return;
      const float lambda = 0.5f;

      vec2 gravity = vec2(0, -g);
      static float t = 0.001f;
      static bool initialized = false;

      if (!initialized) {
         float y0 = spline.r(0).y;
         float y_tau = spline.r(0.001f).y;
         velocity.x = sqrt(2.0f * g * (y0 - y_tau) / (1.0f + lambda));
         initialized = true;
      }

      t += velocity.x * dt;


      vec2 splinePoint2D = spline.r(t);
      vec2 nextPosition = vec2(splinePoint2D.x+radius, splinePoint2D.y);

      float epsilon = 0.001f;
      vec2 tangent = (vec2)normalize(vec2(spline.r(t + epsilon)) - vec2(spline.r(t - epsilon)));

      float tangentialGravity = glm::dot(gravity, tangent);
      if (tangentialGravity < 0&&glm::length(velocity)<0.1f) { // If gravity is pulling in the direction of the tangent
         printf("Switching to free fall!\n");
         // Switch to free fall simulation
         velocity += gravity * dt;  // Simply apply gravity in the free fall mode
         position+=velocity*dt;
         state=fallen;
         return;
      }

      if (state!=fallen) {
         velocity += tangentialGravity * dt;
         position = nextPosition;

         float tangentialVelocity = glm::dot(velocity, tangent);
         float rotationSpeed = tangentialVelocity / radius*0.005f;
         phi -= rotationSpeed * dt;
      }
      if (state == fallen) {
         // Continuously apply gravity in free fall mode
         velocity += gravity * dt; // Gravity is always applied in free fall
         position += velocity * dt; // Update position based on velocity
      }


   }





   void Draw(GPUProgram* gpuProgram) {
      gpuProgram->Use();
      if (state == waiting) return;

      vector<vec2> circleVertices;
      int numSegments = 360;
      circleVertices.push_back(position);

      for (int i = 1; i <= numSegments; i++) {
         float angle = 2.0f * M_PI * i / numSegments;
         circleVertices.push_back(position + radius * vec2(cos(angle), sin(angle)));
      }

      this->Vtx() = circleVertices;
      this->updateGPU();
      Geometry<vec2>::Draw(gpuProgram, GL_TRIANGLE_FAN, vec3(0, 0, 1));

      Geometry<vec2>::Draw(gpuProgram, GL_LINE_LOOP, vec3(1, 1, 1));

      this->Vtx().clear();

      vec2 xAxisLeft  = position + rotate(vec2(-radius, 0), -phi);
      vec2 xAxisRight = position + rotate(vec2(radius, 0), -phi);
      vec2 yAxisBottom = position + rotate(vec2(0, -radius), -phi);
      vec2 yAxisTop = position + rotate(vec2(0, radius), -phi);


      this->Vtx().push_back(xAxisLeft);
      this->Vtx().push_back(xAxisRight);
      this->Vtx().push_back(yAxisBottom);
      this->Vtx().push_back(yAxisTop);

      this->updateGPU();
      Geometry<vec2>::Draw(gpuProgram, GL_LINES, vec3(1, 1, 1));
   }

   vec2 rotate(vec2 point, float angle) {
      float cosA = cos(angle);
      float sinA = sin(angle);
      return vec2(cosA * point.x - sinA * point.y,
                  sinA * point.x + cosA * point.y);
   }
};
``` 


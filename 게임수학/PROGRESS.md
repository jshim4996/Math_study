# Game Mathematics Study Plan (게임 수학 학습 계획)

> Goal: Practically understand and apply math needed for game development
> 목표: 게임 개발에 필요한 수학을 실용적으로 이해하고 적용하는 수준

---

## AI Study Guide (AI 학습 가이드)

### Role (역할)
- AI is a **senior developer and professor** teaching this curriculum
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### Language Rule (언어 규칙)
- **Default: Explain in English (기본: 영어로 설명)**
- When user says "번역" or "translate" → Provide Korean translation
- **Explain using game scenarios** (character movement, bullet firing, etc.)
- Focus on **how to use formulas**, not deriving them

### Study Flow (학습 진행 흐름) - 5 Steps

**Step 1: Check Progress** - "What's next?"
**Step 2: Explanation** - "Explain this" → AI explains with game examples
**Step 3: Example** - "Show example" → Game scenario + code
**Step 4: Assignment** - "Give me an assignment"
**Step 5: Evaluation** - [Submit code/answer] → Pass = [x] check

### Evaluation Criteria (평가 기준)
□ Did you understand and apply the concept in a game context? (게임에 적용했는가)

---

## STEP 1. Math Basics (수학 기초)

### Chapter 01. Coordinate Systems (좌표계)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] What is a Coordinate System (좌표계란 무엇인가)
- [ ] 2D Coordinates (x, y) (2D 좌표계)
- [ ] Screen Coordinates vs Math Coordinates (화면 좌표계 vs 수학 좌표계)
- [ ] 3D Coordinates (x, y, z) (3D 좌표계)
- [ ] Left-handed vs Right-handed Coordinates (왼손 좌표계 vs 오른손 좌표계)

---

### Chapter 02. Basic Operations (기초 연산)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Distance Calculation: Pythagorean Theorem (거리 계산)
- [ ] Distance Between Two Points (두 점 사이의 거리)
- [ ] Midpoint Calculation (중점 계산)
- [ ] Distance in Games: Enemy Detection, Collision Range (게임에서 거리 활용)

---

## STEP 2. Vectors (벡터)

### Chapter 03. Vector Basics (벡터 기초)
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] What is a Vector: Magnitude + Direction (벡터란 무엇인가)
- [ ] Scalar vs Vector (스칼라 vs 벡터)
- [ ] Position Vectors (위치 벡터)
- [ ] Vector Representation: Arrow, Components (벡터 표현)
- [ ] Vectors in Games = Position, Velocity, Direction (게임에서 벡터)

---

### Chapter 04. Vector Operations (벡터 연산)
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] Vector Addition: Movement (벡터 덧셈)
- [ ] Vector Subtraction: Direction Finding (벡터 뺄셈)
- [ ] Scalar Multiplication: Speed Control (스칼라 곱)
- [ ] Vector Magnitude (벡터의 크기)
- [ ] Unit Vectors: Normalization (단위 벡터)

---

### Chapter 05. Vector Applications (벡터 활용)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] Character Movement Implementation (캐릭터 이동 구현)
- [ ] Moving Toward a Target (목표 방향으로 이동)
- [ ] Separating Speed and Direction (속도와 방향 분리)
- [ ] Bullet/Projectile Movement (총알/투사체 이동)

---

### Chapter 06. Dot Product (내적)
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] What is Dot Product (내적이란 무엇인가)
- [ ] Dot Product Formula (내적 공식)
- [ ] Finding Angles with Dot Product (내적으로 각도 구하기)
- [ ] Dot Product Use: Field of View Check (시야각 판정)
- [ ] Dot Product Use: Basic Lighting (조명 계산 기초)

---

### Chapter 07. Cross Product (외적)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] What is Cross Product (3D) (외적이란 무엇인가)
- [ ] Cross Product Formula (외적 공식)
- [ ] Cross Product Result = Perpendicular Vector (외적 결과)
- [ ] Cross Product Use: Left/Right Detection (왼쪽/오른쪽 판정)
- [ ] Cross Product Use: Normal Vectors (법선 벡터)

---

## STEP 3. Trigonometry (삼각함수)

### Chapter 08. Trigonometry Basics (삼각함수 기초)
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] Angle Units: Degrees, Radians (각도 단위)
- [ ] Degrees ↔ Radians Conversion (도 ↔ 라디안 변환)
- [ ] What are sin, cos, tan? (sin, cos, tan 이란?)
- [ ] Right Triangles and Trigonometric Ratios (직각삼각형과 삼각비)
- [ ] Unit Circle and Trigonometric Functions (단위원과 삼각함수)

---

### Chapter 09. Trigonometry Applications (삼각함수 활용)
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] Creating Direction Vector from Angle (각도로 방향 벡터 만들기)
- [ ] Circular Motion Implementation (원 운동 구현)
- [ ] Character Rotation (캐릭터 회전)
- [ ] Firing Bullets at Specific Angles (총알을 특정 각도로 발사)
- [ ] Fan-shaped Area Attacks (부채꼴 범위 공격)

---

### Chapter 10. Inverse Trigonometric Functions (역삼각함수)
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] What is atan2 (atan2란 무엇인가)
- [ ] Getting Angle from Vector (벡터에서 각도 구하기)
- [ ] Look-at Rotation Implementation (타겟을 바라보는 회전 구현)
- [ ] Enemy Rotating Toward Player (적이 플레이어를 향해 회전)

---

## STEP 4. Matrices (행렬)

### Chapter 11. Matrix Basics (행렬 기초)
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] What is a Matrix (행렬이란 무엇인가)
- [ ] Matrix Representation: Rows, Columns (행렬 표현)
- [ ] Matrix Addition, Subtraction (행렬 덧셈, 뺄셈)
- [ ] Matrix Multiplication (행렬 곱셈)
- [ ] Identity Matrix (단위 행렬)

---

### Chapter 12. Transformation Matrices 2D (변환 행렬 2D)
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] What is Transformation (변환이란 무엇인가)
- [ ] Translation (이동 변환)
- [ ] Scale (크기 변환)
- [ ] Rotation (회전 변환)
- [ ] Combining Transformation Matrices (변환 행렬 조합)

---

### Chapter 13. Transformation Matrices 3D (변환 행렬 3D)
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] 3D Translation, Scale, Rotation Matrices (3D 변환 행렬)
- [ ] Homogeneous Coordinates (동차 좌표계)
- [ ] 4x4 Transformation Matrix (4x4 변환 행렬)
- [ ] Transformation Order Matters: TRS (변환 순서의 중요성)

---

### Chapter 14. Coordinate Spaces (좌표 공간)
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] Local vs World Coordinates (로컬 좌표계 vs 월드 좌표계)
- [ ] Model, View, Projection Transforms (모델, 뷰, 투영 변환)
- [ ] MVP Matrix Concept (MVP 행렬)
- [ ] Parent-Child Transforms: Hierarchy (부모-자식 변환)

---

## STEP 5. Collision Detection (충돌 감지)

### Chapter 15. Collision Detection Basics (충돌 감지 기초)
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] What is Collision Detection (충돌 감지란 무엇인가)
- [ ] Point vs Rectangle Collision (점과 사각형 충돌)
- [ ] Point vs Circle Collision (점과 원 충돌)
- [ ] AABB: Axis-Aligned Bounding Box
- [ ] AABB vs AABB Collision

---

### Chapter 16. Circle/Sphere Collision (원/구 충돌)
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] Circle vs Circle Collision (2D) (원 vs 원 충돌)
- [ ] Sphere vs Sphere Collision (3D) (구 vs 구 충돌)
- [ ] Circle vs AABB Collision (원 vs AABB 충돌)

---

### Chapter 17. Advanced Collision Detection (고급 충돌 감지)
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] OBB: Oriented Bounding Box Concept (OBB 개념)
- [ ] Separating Axis Theorem (SAT) Concept (분리 축 정리)
- [ ] Ray Casting (레이캐스팅)
- [ ] Collision Detection Optimization (충돌 감지 최적화)

---

## STEP 6. Curves and Interpolation (곡선과 보간)

### Chapter 18. Interpolation (보간)
> **doc**: `chapters/chapter18/chapter18.md`
> **examples**: `chapters/chapter18/examples/`

- [ ] What is Interpolation (보간이란 무엇인가)
- [ ] Linear Interpolation: Lerp (선형 보간)
- [ ] Smooth Movement Implementation (부드러운 이동 구현)
- [ ] Color Interpolation, Size Interpolation (색상 보간, 크기 보간)
- [ ] Spherical Linear Interpolation: Slerp Concept (구면 선형 보간)

---

### Chapter 19. Curves (곡선)
> **doc**: `chapters/chapter19/chapter19.md`
> **examples**: `chapters/chapter19/examples/`

- [ ] Why We Need Curves (곡선이 필요한 이유)
- [ ] Bezier Curves (베지어 곡선)
- [ ] Quadratic and Cubic Bezier (2차, 3차 베지어)
- [ ] Curved Path Movement (곡선 경로 이동)
- [ ] Animation Curves: Ease In/Out (애니메이션 커브)

---

## STEP 7. Basic Physics (기초 물리)

### Chapter 20. Motion (운동)
> **doc**: `chapters/chapter20/chapter20.md`
> **examples**: `chapters/chapter20/examples/`

- [ ] Velocity (속도)
- [ ] Acceleration (가속도)
- [ ] Uniform Motion, Uniformly Accelerated Motion (등속, 등가속 운동)
- [ ] Implementing Gravity (중력 구현)
- [ ] Implementing Jump (점프 구현)

---

### Chapter 21. Forces and Motion (힘과 운동)
> **doc**: `chapters/chapter21/chapter21.md`
> **examples**: `chapters/chapter21/examples/`

- [ ] Force (힘)
- [ ] Newton's Second Law: F = ma (뉴턴의 제2법칙)
- [ ] Combining Multiple Forces (여러 힘의 합성)
- [ ] Basic Friction (마찰력 기초)
- [ ] Simple Physics Simulation (간단한 물리 시뮬레이션)

---

### Chapter 22. Collision Response (충돌 반응)
> **doc**: `chapters/chapter22/chapter22.md`
> **examples**: `chapters/chapter22/examples/`

- [ ] Reflection After Collision (충돌 후 반사)
- [ ] Reflection Vector Calculation (반사 벡터 계산)
- [ ] Elastic Collision Concept (탄성 충돌 개념)
- [ ] Simple Ball Bouncing Implementation (간단한 공 튕기기 구현)

---

## STEP 8. Probability - Game Systems (확률 - 게임 시스템)

### Chapter 23. Probability Basics (확률 기초)
> **doc**: `chapters/chapter23/chapter23.md`
> **examples**: `chapters/chapter23/examples/`

- [ ] What is Probability (확률이란 무엇인가)
- [ ] Basic Probability Calculation (확률 계산 기초)
- [ ] Independent vs Dependent Events (독립 사건, 종속 사건)
- [ ] Expected Value (기대값)

---

### Chapter 24. Game Probability Systems (게임 확률 시스템)
> **doc**: `chapters/chapter24/chapter24.md`
> **examples**: `chapters/chapter24/examples/`

- [ ] Random vs Probability (랜덤과 확률의 차이)
- [ ] Weighted Random (가중치 랜덤)
- [ ] Gacha/Drop Rate Design (가챠/드롭 확률 설계)
- [ ] Pseudo Random (의사 난수)
- [ ] Pity System (천장 시스템)

---

## Practical Connections (실무 연결 포인트)

| Math Concept | Game Application |
|--------------|-----------------|
| Vectors | Movement, Direction, Velocity |
| Dot Product | Field of view, Front/Back detection |
| Cross Product | Left/Right detection, Normals |
| Trigonometry | Rotation, Circular motion, Firing angles |
| Matrices | Transforms, Coordinate systems, Camera |
| Collision Detection | Physics, Interactions |
| Interpolation | Smooth movement, Animation |
| Physics | Jump, Gravity, Projectiles |
| Probability | Drops, Gacha, Random events |

---

## Study Tips (학습 팁)

```
1. Don't memorize formulas - understand when to use them
2. Implement directly in games
3. Use built-in functions: Vector3.Dot, Lerp, etc.
4. Visually verify results (debug draw)
```

---

## For Later (나중에 필요할 때)

| Topic | When |
|-------|------|
| Quaternions | Deep understanding of 3D rotation |
| Inverse Matrices | Inverse coordinate transforms |
| Calculus | Physics engine development |
| Shader Math | Graphics programming |
| Noise Functions (Perlin) | Procedural generation |

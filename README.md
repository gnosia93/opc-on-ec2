# opc-on-ec2
* Optical Proximity Correction

워크샵 명칭: "Slurm 기반 대규모 이미지 연산 최적화: OPC 워크로드 시뮬레이션"

### [Step 1] 워크로드 설계: 이미지 타일링 및 복잡 연산 (Python) ###
* 단순한 필터가 아니라, OPC 특유의 반복적 수치 해석 느낌을 주는 스크립트를 작성합니다.
* 실습 내용: 50,000 x 50,000 픽셀 이상의 대형 바이너리 이미지를 128x128 타일로 쪼개고, 각 타일에 대해 FFT(고속 푸리에 변환) 또는 고차원 컨볼루션 연산을 수행합니다.
* 핵심 도구: NumPy, OpenCV, Multiprocessing 라이브러리.
* 개발자 미션: "타일 간 경계(Boundary) 데이터를 어떻게 공유하며 연산 정확도를 유지할 것인가?"

### [Step 2] 프로파일링: CPU 유닛 점유율 분석 ###
* 인프라를 늘리기 전, 단일 노드 내에서의 최적화를 먼저 배웁니다.
* 실습 내용: Linux perf를 이용해 cache-misses, instructions-per-cycle(IPC), branch-misses 등을 측정합니다.
* 핵심 포인트:
* Python의 GIL(Global Interpreter Lock)이 멀티코어 성능을 어떻게 방해하는지 눈으로 확인.
* Numpy/MKL 라이브러리가 CPU의 AVX-512 명령어를 제대로 쓰고 있는지 확인하여 '수학 연산 가속'의 중요성 체득.

### [Step 3] 분산 실행 및 Tail Latency 트러블슈팅 ###
* 이제 Slurm 클러스터에 수백 개의 Job으로 배포합니다.
* 실습 내용: sbatch와 srun을 사용해 수백 대의 노드에 태스크를 뿌립니다.
* 의도적 장애(Chaos Engineering): 일부 노드에 의도적으로 CPU 부하를 주거나 I/O 대역폭을 제한하여 Tail Latency를 발생시킵니다.
* 핵심 포인트:
* 전체 작업의 99%가 끝나도 1%의 느린 노드(Straggler) 때문에 전체 마스크 생성이 늦어지는 현상 경험.
* Slurm의 Preemption(우선순위 기반 작업 회수)과 Re-queue(재시도) 기능을 이용해 이를 극복하는 아키텍처 구현.

---
* 1. 전처리 및 데이터 파티셔닝 (Tiling & Fragmentation)
거대한 설계 도면(GDSII/OASIS 파일)을 CPU 수천 개가 나눠 가질 수 있게 쪼개는 단계입니다.
워크로드 특징: 연산보다는 I/O와 메모리 집약적입니다.
워크샵 포인트:
데이터를 너무 작게 쪼개면 오버헤드가 커지고, 너무 크게 쪼개면 연산 시간이 불균형(Skew)해지는 현상 분석.
Boundary 관리: 잘린 타일의 경계면에 있는 패턴을 어떻게 연속성 있게 처리할 것인가에 대한 알고리즘적 이해.

* 2. 계산 모델링 (Optical & Resist Simulation)
빛의 회절과 포토레지스트의 반응을 수학적으로 모델링하여 시뮬레이션하는 단계입니다.
워크로드 특징: CPU 연산(Floating Point) 집약적입니다.
워크샵 포인트:
Vectorization: AVX-512 같은 CPU 명령어 세트가 수치 계산 속도에 미치는 영향.
Scalability: 코어 수를 늘릴 때 성능이 선형적으로 증가하는지, 아니면 특정 지점에서 Amdahl의 법칙에 걸리는지 추적.

* 3. 일관성 유지와 결과 병합 (Merging & Verification)
각 노드에서 계산된 타일들을 다시 하나의 마스크 데이터로 합치는 과정입니다.
워크로드 특징: 네트워크 대역폭과 공유 스토리지 성능이 핵심입니다.
워크샵 포인트:
수천 대의 노드가 연산을 끝내고 동시에 파일을 쓸 때 발생하는 Storage Hotspot 해결 전략.
Amazon FSx for Lustre의 분산 성능을 활용한 병합 시간 단축 실습

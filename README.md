# opc-on-aws


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

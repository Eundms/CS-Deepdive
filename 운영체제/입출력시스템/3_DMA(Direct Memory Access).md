# DMA
CPU 개입 없이 I/O 장치가 메모리에 직접 데이터를 전송할 수 있는 기술   
시스템 성능을 향상시키는 중요한 기법 중 하나  

## CPU를 통한 데이터 전송의 문제점
- CPU가 I/O 장치와 메모리 간의 데이터 전송을 직접 처리하면, CPU가 모든 데이터를 읽고 쓰는 과정에 관여해야 함 
CPU가 입출력 처리에 많은 시간을 소몸하여, 다른 작업 수행이 어려움 
특히, 대량의 데이터 전송시 CPU 부하가 급증하여 전체 시스템 성능이 저하됨 

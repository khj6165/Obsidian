![[Pasted image 20241125114621.png]]

API 
https://cloud.google.com/bigquery/docs/reference/libraries-overview?hl=ko

## BigQuery API

데이터 세트, 테이블, 작업, 루틴과 같은 핵심 리소스를 만들고 수정하고 삭제하기 위한 리소스를 제공하는 기본 API입니다.
## BigQuery Data Policy API

이 API를 사용하면 사용자가 열 수준 보안 및 데이터 마스킹을 위한 BigQuery 데이터 정책을 관리할 수 있습니다.
## BigQuery Migration API

이 API는 사용자가 기존 데이터 웨어하우스를 BigQuery로 마이그레이션할 수 있게 해주는 메커니즘을 지원합니다. 주로 SQL 변환과 같이 처리할 일련의 워크플로 및 태스크로 작업을 모델링합니다.
## BigQuery Storage API

이 API는 자체 애플리케이션 및 도구에서 대량의 관리 데이터를 검색해야 하는 소비자를 위해 처리량이 높은 데이터 읽기를 제공합니다. 이 API는 스토리지를 스캔하는 병렬 메커니즘을 지원하고 열 프로젝트 및 필터링과 같은 기능을 활용하기 위한 지원을 제공합니다.
## BigQuery Reservation API

이 API는 엔터프라이즈 사용자가 슬롯 및 BigQuery BI Engine 메모리 할당과 같은 전용 리소스를 프로비저닝하고 관리할 수 있는 메커니즘을 제공합니다.
## Analytics Hub

이 API는 조직 내부 및 조직 간 데이터 공유를 용이하게 합니다. 데이터 제공업체가 공유 BigQuery 데이터 세트를 참조하는 목록을 게시할 수 있습니다. Analytics Hub를 사용하여 사용자가 액세스할 수 있는 목록을 탐색하고 검색할 수 있습니다. 구독자가 목록을 보고 구독할 수 있습니다. 목록을 구독하면 Analytics Hub가 프로젝트에 연결된 데이터 세트를 만듭니다.
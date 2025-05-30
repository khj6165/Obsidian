
 gcp_manager@sedp.co.kr / voTmdnjem1!
llion-control-master@lottemart-dw.iam.gserviceaccount.com


## IAM에서 Cloud Billing 역할 개요

사용자에게 권한을 직접 부여하는 대신 하나 이상의 권한이 번들로 포함된 _역할_을 부여합니다.

동일한 사용자 또는 동일한 리소스에 하나 이상의 역할을 부여할 수 있습니다.

다음과 같은 사전 정의된 Cloud Billing IAM 역할은 액세스 제어를 사용하여 업무 분리를 시행할 수 있도록 설계되었습니다.

| 역할                                                | 목적                                            | 수준             | 사용 사례                                                                                                                                                                                                                |
| ------------------------------------------------- | --------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 결제 계정 생성자  <br>(`roles/billing.creator`)          | 새로운 셀프 서비스(온라인) 결제 계정을 만듭니다.                  | 조직             | 이 역할은 초기 결제 설정을 지정하거나 추가 결제 계정을 만들 때 사용됩니다.  <br>이 역할이 있는 사용자만 회사 ID를 사용하여 신용카드로 Google Cloud에 가입할 수 있습니다.  <br>**팁**: 조직 내에서 추적할 수 없는 클라우드 지출 확산을 방지할 수 있도록 이 역할이 있는 사용자 수를 최소화하세요.                                 |
| 결제 계정 관리자  <br>(`roles/billing.admin`)            | 결제 계정을 관리합니다(만들기 제외).                         | 조직 또는 결제 계정    | 이 역할은 결제 계정의 소유자 역할입니다. 이 역할을 사용하여 결제 수단을 관리하고, 결제 내보내기를 구성하고, 비용 정보를 확인하고, 프로젝트를 연결 및 연결 해제하고, 결제 계좌에 대한 다른 사용자 역할을 관리합니다. 기본적으로 Cloud Billing 계정을 만드는 사람은 Cloud Billing 계정의 `Billing Account Administrator`입니다.    |
| 결제 계정 비용 관리자  <br>(`roles/billing.costsManager`)  | 예산을 관리하고 결제 계정의 비용 정보를 보고 내보냅니다(가격 책정 정보 제외). | 조직 또는 결제 계정    | 예산을 생성, 수정, 삭제하고, 결제 계정 비용 정보 및 거래를 확인하고, BigQuery로 결제 비용 데이터 내보내기를 관리합니다. 가격 책정 페이지에서 _가격 책정_ 데이터를 내보내거나 _커스텀 가격 책정_을 볼 수 있는 권한을 부여하지 않습니다. 또한 프로젝트 연결 또는 연결 해제 또는 결제 계정의 속성 관리가 허용되지 않습니다.                         |
| 결제 계정 뷰어  <br>(`roles/billing.viewer`)            | 결제 계정 비용 정보와 거래를 확인합니다.                       | 조직 또는 결제 계정    | 결제 계정 뷰어 액세스 권한은 일반적으로 재무팀에 부여되며, 지출 정보에 대한 액세스 권한을 제공하지만, 프로젝트를 연결 또는 연결 해제하거나 결제 계정의 속성을 관리할 수 있는 권한을 부여하지 않습니다.                                                                                                   |
| 결제 계정 사용자  <br>(`roles/billing.user`)             | 프로젝트를 결제 계정에 연결합니다.                           | 조직 또는 결제 계정    | 이 역할은 매우 제한적인 권한을 가지므로, 광범위하게 부여할 수 있습니다. 두 가지 역할을 프로젝트 생성자와 함께 사용하면 사용자가 결제 계정 사용자 역할이 부여된 결제 계정에 연결된 새 프로젝트를 만들 수 있습니다. 또는 두 역할을 프로젝트 결제 관리자 역할과 함께 사용하면 사용자가 결제 계정 사용자 역할이 부여된 결제 계정에서 프로젝트를 연결하거나 연결 해제할 수 있습니다. |
| 프로젝트 결제 관리자  <br>(`roles/billing.projectManager`) | 결제 계정에 대해 프로젝트를 연결/연결 해제합니다.                  | 조직, 폴더 또는 프로젝트 | _결제 계정 사용자_ 역할과 함께 사용되는 경우 프로젝트 결제 관리자 역할은 사용자가 결제 계정에 프로젝트를 연결할 수 있도록 허용하지만 리소스에 대한 권한을 부여하지 않습니다. 프로젝트 소유자는 이 역할을 사용하여 리소스 액세스 권한을 부여하지 않고도 다른 사람이 프로젝트 결제를 관리하도록 허용할 수 있습니다.                                      |

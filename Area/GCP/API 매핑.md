### API 목록
https://cloud.google.com/apis?hl=ko

### API 액세스
[https://developers.google.com/identity/protocols/oauth2/web-server?hl=ko](https://developers.google.com/identity/protocols/oauth2/web-server?hl=ko#httprest_1)

스프링클라우드 GCP https://spring.io/projects/spring-cloud-gcp#overview
https://googleapis.dev/java/spring-cloud-gcp/5.8.0/index.html
### 관리도구 API
##### 공개 서비스 및 가격 정보 가져오기
1. Billing Budget API : 결제계정ID와 프로젝트ID를 통해 **예산**을 가져온다.  https://cloud.google.com/billing/docs/how-to/budget-api?hl=ko
2. 모든 공개 Google Cloud SKU의 가격 책정 정보 가져오기
##### Cloud Billing 계정 관련 정보 가져오기
1. Cloud Billing 계정의 SKU 그룹 목록 가져오기
2. Cloud Billing 계정의 SKU 그룹에 SKU 목록 가져오기
3. **Cloud Billing 계정에 대해 모든 Google Cloud SKU의 가격 가져오기** : https://cloud.google.com/billing/docs/how-to/get-pricing-information-api?hl=ko#get_the_prices_for_all_skus_for_your_account
	GET https://cloudbilling.googleapis.com/v1beta/billingAccounts/BILLING_ACCOUNT_ID/skus/SKU_ID/prices?pageSize=PAGE_SIZE&pageToken=PAGE_TOKEN
	- SKU_ID: 가격을 가져올 특정 SKU ID입니다. 모든 SKU의 가격을 가져오려면 `-`을 사용합니다.
6. Cloud Billing 계정의 SKU 가격 가져오기 
	-  이 API를 호출하려면 Cloud Billing 계정에 대한 `billing.billingAccountPrice.get` IAM 권한이 필요합니다.
	2- GET https://cloudbilling.googleapis.com/v1beta/billingAccounts/BILLING_ACCOUNT_ID/skus/SKU_ID/price?currencyCode=CURRENCY


### 보안 및 ID API

#### API access

IAM supports programmatic access. You can access the API in the following ways:

- [Client libraries](https://cloud.google.com/iam/docs/authentication#client-libraries)
- [Google Cloud CLI](https://cloud.google.com/iam/docs/authentication#gcloud)
- [REST](https://cloud.google.com/iam/docs/authentication#rest)

##### IAM 클라이언트 라이브러리
https://cloud.google.com/iam/docs/reference/libraries

Use the IAM v1 API to manage custom roles, service accounts, and service account keys.

```xml
<dependency>  
	<groupId>com.google.apis</groupId>  
	<artifactId>google-api-services-iam</artifactId>  
	<version>v1-rev20240118-2.0.0</version>
</dependency>
```

Use the IAM v2 API to manage [deny policies](https://cloud.google.com/iam/docs/deny-overview).

```xml
<dependency>  
	<groupId>com.google.cloud</groupId>  
	<artifactId>google-iam-policy</artifactId>  
	<scope>compile</scope>
</dependency>
```

Use the Service Account Credentials API to create short-lived, limited-privilege credentials for service accounts.
```xml
<dependency>  
	<groupId>com.google.apis</groupId>  
	<artifactId>google-api-services-iamcredentials</artifactId>  
	<version>v1-rev20211203-2.0.0</version>
</dependency>
```

- 거부 정책을 만들려면 먼저 거부할 권한과 이러한 권한을 거부할 주 구성원을 결정해야 한다.
- 거부정책에서 지원되는 권한 : https://cloud.google.com/iam/docs/deny-permissions-support?authuser=6
1. 프로젝트 ID로 IAM 정책 가져오기 https://cloud.google.com/iam/docs/samples/iam-get-policy?hl=ko&authuser=6
2. 프로젝트 ID로 리소스 거부 정책 리스트 가져오기 https://cloud.google.com/iam/docs/samples/iam-list-deny-policy?hl=ko&authuser=6
3. 프로젝트ID, 정책ID로 거부정책 상세보기 https://cloud.google.com/iam/docs/samples/iam-get-deny-policy?hl=ko&authuser=6
4. 프로젝트 아이디로 역할 리스트 가져오기 https://cloud.google.com/iam/docs/samples/iam-list-roles?hl=ko&authuser=6
5. 역할ID로 역할 가져오기 https://cloud.google.com/iam/docs/samples/iam-get-role?hl=ko&authuser=6


### BigQuery

![[Pasted image 20241125114621.png]]

API 
https://cloud.google.com/bigquery/docs/reference/libraries-overview?hl=ko
```gradle
implementation(platform("com.google.cloud:libraries-bom:  26.51.0"))  
implementation("com.google.cloud:google-cloud-bigquery")
```

**1. 테이블의 IAM 정책 만들기** https://cloud.google.com/bigquery/docs/samples/bigquery-create-iam-policy?hl=ko&authuser=00
리소스의 IAM 정책을 수정하려면 다음 권한이 필요합니다.

- `` `bigquery.datasets.get` `` - 데이터 세트의 액세스 정책을 가져옵니다.
- `` `bigquery.datasets.update` `` - 데이터 세트의 액세스 정책을 설정합니다.
- `` `bigquery.datasets.getIamPolicy` `` - 데이터 세트의 액세스 정책을 가져옵니다(Google Cloud 콘솔 전용).
- `` `bigquery.datasets.setIamPolicy` `` - 데이터 세트의 액세스 정책을 설정합니다(콘솔 전용).
- `` `bigquery.tables.getIamPolicy` `` - 테이블 또는 뷰의 액세스 정책을 가져옵니다.
- `` `bigquery.tables.setIamPolicy` `` - 테이블 또는 뷰의 액세스 정책을 설정합니다.
2. 테이블 또는 뷰의 액세스 정책 보기 :  [`tables.getIamPolicy` 메서드](https://cloud.google.com/bigquery/docs/reference/rest/v2/tables/getIamPolicy?authuser=00&hl=ko)를 호출합니다.
3. 테이블 또는 뷰에 액세스 권한 부여
```java
import com.google.cloud.bigquery.Acl;
import com.google.cloud.bigquery.Acl.Role;
import com.google.cloud.bigquery.Acl.User;
import com.google.cloud.bigquery.BigQuery;
import com.google.cloud.bigquery.BigQueryException;
import com.google.cloud.bigquery.BigQueryOptions;
import com.google.cloud.bigquery.Dataset;
import java.util.ArrayList;

public class UpdateDatasetAccess {

  public static void main(String[] args) {
    // TODO(developer): Replace these variables before running the sample.
    String datasetName = "MY_DATASET_NAME";
    // Create a new ACL granting the READER role to "sample.bigquery.dev@gmail.com"
    // For more information on the types of ACLs available see:
    // https://cloud.google.com/storage/docs/access-control/lists
    Acl newEntry = Acl.of(new User("sample.bigquery.dev@gmail.com"), Role.READER);

    updateDatasetAccess(datasetName, newEntry);
  }

  public static void updateDatasetAccess(String datasetName, Acl newEntry) {
    try {
      // Initialize client that will be used to send requests. This client only needs to be created
      // once, and can be reused for multiple requests.
      BigQuery bigquery = BigQueryOptions.getDefaultInstance().getService();

      Dataset dataset = bigquery.getDataset(datasetName);

      // Get a copy of the ACLs list from the dataset and append the new entry
      ArrayList<Acl> acls = new ArrayList<>(dataset.getAcl());
      acls.add(newEntry);

      bigquery.update(dataset.toBuilder().setAcl(acls).build());
      System.out.println("Dataset Access Control updated successfully");
    } catch (BigQueryException e) {
      System.out.println("Dataset Access control was not updated \n" + e.toString());
    }
  }
}
```


4. 테이블 또는 뷰에 대한 액세스 권한 취소 
```java
import com.google.cloud.Identity;
import com.google.cloud.Policy;
import com.google.cloud.Role;
import com.google.cloud.bigquery.BigQuery;
import com.google.cloud.bigquery.BigQueryException;
import com.google.cloud.bigquery.BigQueryOptions;
import com.google.cloud.bigquery.TableId;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

// Sample to update iam policy in table
public class UpdateIamPolicy {

  public static void main(String[] args) {
    // TODO(developer): Replace these variables before running the sample.
    String datasetName = "MY_DATASET_NAME";
    String tableName = "MY_TABLE_NAME";
    updateIamPolicy(datasetName, tableName);
  }

  public static void updateIamPolicy(String datasetName, String tableName) {
    try {
      // Initialize client that will be used to send requests. This client only needs to be created
      // once, and can be reused for multiple requests.
      BigQuery bigquery = BigQueryOptions.getDefaultInstance().getService();

      TableId tableId = TableId.of(datasetName, tableName);

      Policy policy = bigquery.getIamPolicy(tableId);
      Map<Role, Set<Identity>> binding = new HashMap<>(policy.getBindings());
      binding.remove(Role.of("roles/bigquery.dataViewer"));

      policy.toBuilder().setBindings(binding).build();
      bigquery.setIamPolicy(tableId, policy);

      System.out.println("Iam policy updated successfully");
    } catch (BigQueryException e) {
      System.out.println("Iam policy was not updated. \n" + e.toString());
    }
  }
}
```
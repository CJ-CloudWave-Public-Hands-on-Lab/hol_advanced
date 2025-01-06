
#### us-east-1 리전에 버킷 생성하기

- `ACCOUNT_ID` 값을 변수에 할당

  ```bash
  ACCOUNT_ID=$(aws sts get-caller-identity | jq -r .Account)
  ```

- 생성할 버킷 이름을 `BUCKET_NAME` 변수에 할당

  ```bash
  BUCKET_NAME="lab-edu-bucket-cloudfront-$ACCOUNT_ID"
  ```

- 버킷 생성

  ```bash
  aws s3 mb s3://$BUCKET_NAME --region us-east-1
  ```

#### 버킷 정책 수정

- Public Access Block 비활성화

  ```bash
  aws s3api put-public-access-block --bucket $BUCKET_NAME --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
  ```

- ACL 정책 활성화

  ```bash
  aws s3api put-bucket-ownership-controls --bucket $BUCKET_NAME --ownership-controls Rules=[{ObjectOwnership=ObjectWriter}]
  ```

#### Sample Data 생성 

- Sample Data 파일 이름 변수 설정

  ```bash
  OBJ_NAME=sample_object
  ```

- 100MB 사이즈의 파일 생성

  ```bash
  dd if=/dev/urandom of=$OBJ_NAME bs=1M count=100
  ```

- Sample Data S3 업로드

  ```bash
  aws s3 cp ./sample_object s3://$BUCKET_NAME --acl public-read
  ```

#### Sample Data CloudFront 캐시 속도 비교

- Object HTTPS URL 변수 설정

  ```bash
  OBJ_URL="https://$BUCKET_NAME.s3.amazonaws.com/$OBJ_NAME"
  ```

- Bucket Object 다운로드 속도 측정

  ```bash
  for i in `seq 1 10`; do echo $i; curl -s -o /dev/null --write-out "size_download: %{size_download} // time_total: %{time_total} // time_starttransfer: %{time_starttransfer}\n" $OBJ_URL; done
  1
  size_download: 104857600 // time_total: 7.423171 // time_starttransfer: 0.667527
  2
  size_download: 104857600 // time_total: 7.644665 // time_starttransfer: 0.559782
  3
  size_download: 104857600 // time_total: 6.859081 // time_starttransfer: 0.590344
  4
  size_download: 104857600 // time_total: 6.404214 // time_starttransfer: 0.549187
  5
  size_download: 104857600 // time_total: 7.051656 // time_starttransfer: 0.585859
  6
  size_download: 104857600 // time_total: 7.075964 // time_starttransfer: 0.597612
  7
  size_download: 104857600 // time_total: 6.379377 // time_starttransfer: 0.537589
  8
  size_download: 104857600 // time_total: 6.315437 // time_starttransfer: 0.573412
  9
  size_download: 104857600 // time_total: 6.612966 // time_starttransfer: 0.556621
  10
  size_download: 104857600 // time_total: 6.555135 // time_starttransfer: 0.545183
  ```

- 결과 확인


- CloudFront Endpoint로 다운로드 속도 측정

  ```bash
  for i in `seq 1 10`; do echo $i; curl -s -o /dev/null --write-out "size_download: %{size_download} // time_total: %{time_total} // time_starttransfer: %{time_starttransfer}\n" https://ddrdtk8lpdh5o.cloudfront.net/sample_object; done
  1
  size_download: 104857600 // time_total: 6.121740 // time_starttransfer: 0.569941
  2
  size_download: 104857600 // time_total: 0.414024 // time_starttransfer: 0.030523
  3
  size_download: 104857600 // time_total: 0.282488 // time_starttransfer: 0.027952
  4
  size_download: 104857600 // time_total: 0.315421 // time_starttransfer: 0.025287
  5
  size_download: 104857600 // time_total: 0.241643 // time_starttransfer: 0.025006
  6
  size_download: 104857600 // time_total: 0.516103 // time_starttransfer: 0.028966
  7
  size_download: 104857600 // time_total: 0.290497 // time_starttransfer: 0.025267
  8
  size_download: 104857600 // time_total: 0.245923 // time_starttransfer: 0.024709
  9
  size_download: 104857600 // time_total: 0.265789 // time_starttransfer: 0.024924
  10
  size_download: 104857600 // time_total: 0.250480 // time_starttransfer: 0.026514
  ```


#### Bucket Web Hosting 설정해서 HTTPS 설정하는 과정 실습
- Web Hosting 해서 Route 53으로 도메인을 연결해도 사용자 정의 도메인 설정을 가능한 것을 보여주고
- HTTPS를 설정하기 위해 ACM을 연동 설정하는게 안되는 것을 보여주기
- 추가로 CloudFront에는 ACM, Route53, WAF를 연동해서 보안이 강화 되는 것을 보여주기
#### OAC 이용해서 Bucket 접근 제어 설정하고, Bucket에는 CloudFront에서만 접근되는 것을 확인 + 보안성 강화되는 것을 보여주기
#### HTTP to HTTPS Redirect 설정 
- `curl -v -o /dev/null http://[CloudFront endpoint]` 명령을 통해서 Resoponse 데이터가 바뀌는 것을 보여주기
#### Custom Error Page 연동 설정
#### Edge Function 이용


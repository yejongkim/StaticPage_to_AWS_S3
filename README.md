# StaticPage to AWS S3
20260213
 

### 1단계: AWS S3 버킷 설정.

 **S3 버킷을 생성하고 권한을 열어둬야 합니다.**

1. **버킷 생성:** (예: `my-react-library-2024`)
2. **속성:** 정적 웹 호스팅 활성화.
3. **권한 (퍼블릭 액세스 차단):** 모든 차단 해제.
4. **권한 (버킷 정책):** 아래 정책 붙여넣기.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::내-버킷-이름/*"
        }
    ]
}

```

---
 

### 2단계: AWS Academy에서 임시 자격 증명 확보

1. AWS Academy Learner Lab에 접속하여 **[Start Lab]**을 누르고 상태가 초록색 점(`Ready`)이 될 때까지 기다립니다.
2. 상단 메뉴의 **[AWS Details]**를 클릭합니다.
3. **AWS CLI** 섹션의 `Show` 버튼을 누르면 아래 3가지 값을 볼 수 있습니다.
* `aws_access_key_id`
* `aws_secret_access_key`
* `aws_session_token` (이게 가장 중요합니다!)

AWS Academy(Learner Lab) 환경에서는 보안상 **IAM 사용자(User)를 새로 만들 수 없도록 막혀있는 것**이 정상입니다. 대신 AWS Academy가 제공하는 **"임시 자격 증명(Temporary Credentials)"**을 사용해야 합니다.

이 방식은 일반적인 운영 환경과 달리 **`AWS_SESSION_TOKEN`**이라는 것이 추가로 필요하며, **일정 시간(보통 3~4시간)마다 만료**되어 GitHub Secrets를 갱신해줘야 한다는 번거로움이 있지만, 현재 환경에서 CI/CD를 구축할 수 있는 유일한 방법입니다.
  


### 3단계: GitHub Secrets 등록 (Academy 토큰 3개)

Academy 환경이므로 세션 토큰이 포함된 3가지 값을 GitHub 저장소 **Settings > Secrets and variables > Actions**에 등록합니다. (세션이 만료되었다면 [AWS Details]에서 다시 복사해야 합니다!)

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_SESSION_TOKEN` (필수!)

---



  
### 4단계: GitHub Actions 워크플로우 작성 (`deploy.yml`)

임시 자격 증명을 사용하려면 `.yml` 파일에서 **Session Token**을 인식하도록 수정해야 합니다.  

```yaml
name: Deploy to AWS S3 (Academy)

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }} # 이 줄이 반드시 추가되어야 합니다!
        aws-region: us-east-1 # Academy는  'us-east-1', 'us-west-1'만 허용됩니다. 

    - name: Deploy to S3
      run: |
        aws s3 sync ./ s3://내-버킷-이름 --delete --exclude ".git/*" --exclude ".github/*"

```

> **주의: 리전(Region) 설정**
> AWS Academy는 대부분 **미국 버지니아북부 (us-east-1), 미국 오레곤(us-west-1)** 리전만 사용 가능하게 제한되어 있습니다. `aws-region`을 `us-east-1`로 설정하고, S3 버킷도 해당 리전에 생성했는지 꼭 확인하세요.

---
  

### 5단계: 배포 및 확인

1. 코드를 GitHub에 `push` 합니다.
2. **Actions** 탭에서 `Build` 단계와 `Deploy` 단계가 성공하는지 확인합니다.
3. S3 버킷의 **정적 웹 사이트 호스팅 엔드포인트** 주소로 접속합니다.
 
 

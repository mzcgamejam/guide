# GameLift, Unity로 세션 생성, 입장, 가위바위보 게임 만들기

---
MEGAZONE CLOUD
---
## Overview

GameLift와 API Gateway, Lambda, SQS, RDS를 통해 Session을 만들고 입장하여 가위, 바위, 보 게임을 진행하고 결과를 저장해봅니다.

---

## 목표

GameLift의 기본적인 Build, Fleet 생성 및 구조 이해. 

---

## 장점

![readme/0001.png](readme/0001.png)

GameLift 는 Session 게임에 특화된 서비스 입니다. 시작과 끝이 명확한 Session 형 게임을 FleetIQ를 이용하여 Spot 인스턴스를 활용하고 비용을 절감합니다.

또한 Session 형 게임 서버의 손쉬운 Auto-scaling 및 Matchmaking을 지원하고 있습니다. 

---

## Architecture

![readme/Infra-001.png](readme/Infra-001.png)

---

## Step 1. 설치

### Step 1.1. Unity Hub 설치

여러 버전의 Unity를 설치하고 관리할 수 있는 [Unity Hub를 설치](https://unity3d.com/kr/get-unity/download)합니다.

![readme/_1.png](readme/_1.png)

Unity Hub를 실행한 후 왼쪽 설치 탭으로 이동합니다.

![readme/Untitled.png](readme/Untitled.png)

프로젝트에서 사용한 2020.1.11f1 버전을 설치합니다.

![readme/Untitled%201.png](readme/Untitled%201.png)

원하는 버전이 목록에 없다면 [Unity 다운로드 아카이브](https://unity3d.com/get-unity/download/archive)로 이동합니다.

![readme/Untitled%202.png](readme/Untitled%202.png)

원하는 버전을 찾습니다.

![readme/Untitled%203.png](readme/Untitled%203.png)

Unity Hub 버튼을 누르시면 설치가 진행됩니다. 

"Unity Hub를 여시겠습니까?" 라는 팝업이 열리시면 Unity Hub 열기 버튼을 누르시면 됩니다.

![readme/Untitled%204.png](readme/Untitled%204.png)

기본적으로 Android Build Support와 Windows Build Support(IL2CPP)가 선택 되어 있습니다.(Windows 기준), 추가로 iOS Build Support 등을 선택 하실 수 있습니다. 또한 Visual Studio Community 2019 설치 또한 기본 선택되어 있습니다. 없으시다면 함께 INSTALL 하시고 설치되어 있으시다면 **체크 해제**하시면 됩니다.

![readme/Untitled%205.png](readme/Untitled%205.png)

### Step 1.2. AWS Toolkit for Visual Studio 설치

Visual Studio 2019를 실행하고 AWS Toolkit for Visual Studio 를 설치합니다.

![readme/lambda-001.png](readme/lambda-001.png)

![readme/lambda-002.png](readme/lambda-002.png)

![readme/lambda-003.png](readme/lambda-003.png)

![readme/lambda-004.png](readme/lambda-004.png)

![readme/lambda-005.png](readme/lambda-005.png)

![readme/lambda-006.png](readme/lambda-006.png)

![readme/lambda-007.png](readme/lambda-007.png)

## Step 2. 프로젝트 가져오기

### Step 2.1. Server

프로젝트 : [https://github.com/mzcgamejam/megajam_sample_2022/tree/main/ctc-game-server](https://github.com/mzcgamejam/megajam_sample_2022/tree/main/ctc-game-server)


### Step 2.2. Client

프로젝트 : [https://github.com/mzcgamejam/megajam_sample_2022/tree/main/ctc-game-client](https://github.com/mzcgamejam/megajam_sample_2022/tree/main/ctc-game-client)

## Step 3. 네트워크 생성

### Step 3.1. VPC 생성

VPC 대시보드로 이동하여 VPC 생성을 아래와 같이 진행합니다.

![readme/Untitled%206.png](readme/Untitled%206.png)

### Step 3.2. Subnet 생성

2개의 퍼블릭 서브넷, 2개의 프라이빗 서브넷 생성을 아래와 같이 진행합니다.

![readme/VPC-002.png](readme/VPC-002.png)

![readme/Untitled%207.png](readme/Untitled%207.png)

![readme/Untitled%208.png](readme/Untitled%208.png)

![readme/Untitled%209.png](readme/Untitled%209.png)

![readme/Untitled%2010.png](readme/Untitled%2010.png)

### Step 3.3. 인터넷 게이트웨이 생성 및 연결

Internet Gateway를 생성하고 VPC에 연결하는 작업을 진행합니다.
VPC 대시보드에서 **인터넷 게이트웨이**를 클릭 후, **인터넷 게이트웨이 생성** 버튼을 클릭합니다.

![readme/Untitled%2011.png](readme/Untitled%2011.png)

![readme/Untitled%2012.png](readme/Untitled%2012.png)

인터넷 게이트웨이를 VPC에 연결합니다.

다음으로 생성한 인터넷 게이트웨이를 VPC에 연결하는 작업을 진행합니다.

**mzc-game-ig**를 선택 후, 작업 탭을 확장하여 **VPC에 연결**을 클릭합니다.

![readme/Untitled%2013.png](readme/Untitled%2013.png)

**VPC 연결** 설정창에서 생성한 mzc-game-vpc를 선택 후, **인터넷 게이트웨이 연결** 버튼을 클릭합니다.

![readme/Untitled%2014.png](readme/Untitled%2014.png)

### Step 3.4. NAT 게이트웨이 생성

![readme/NATGateway-001.png](readme/NATGateway-001.png)

![readme/NATGateway-002.png](readme/NATGateway-002.png)

### Step 3.5. 라우팅 테이블 생성 및 설정

라우팅 테이블 생성 및 설정을 진행합니다.

**라우팅 테이블 생성** 버튼을 클릭합니다.

![readme/Untitled%2015.png](readme/Untitled%2015.png)

먼저 퍼블릭 라우팅 테이블을 생성합니다.

![readme/Untitled%2016.png](readme/Untitled%2016.png)

![readme/Untitled%2017.png](readme/Untitled%2017.png)

![readme/Untitled%2018.png](readme/Untitled%2018.png)

![readme/Untitled%2019.png](readme/Untitled%2019.png)

![readme/Untitled%2020.png](readme/Untitled%2020.png)

프라이빗 라우팅 테이블을 생성합니다.

![readme/RT-007.png](readme/RT-007.png)

![readme/Untitled%2021.png](readme/Untitled%2021.png)

![readme/Untitled%2022.png](readme/Untitled%2022.png)

![readme/Untitled%2023.png](readme/Untitled%2023.png)

![readme/Untitled%2024.png](readme/Untitled%2024.png)

## Step 4. 보안 그룹 만들기

### Step 4.1. Lambda 에서 사용할 보안 그룹 만들기

![readme/Untitled%2025.png](readme/Untitled%2025.png)

![readme/SecurityGroup-002.png](readme/SecurityGroup-002.png)

![readme/Untitled%2026.png](readme/Untitled%2026.png)

### Step 4.2. RDS 에서 사용할 보안 그룹 생성

![readme/Untitled%2027.png](readme/Untitled%2027.png)

![readme/Untitled%2028.png](readme/Untitled%2028.png)

![readme/Untitled%2029.png](readme/Untitled%2029.png)

## Step 5. IAM 설정

### Step 5.1. GameLift 정책 만들기

이 후 만들 역할에서 사용할 정책을 GameLift 정책을 생성 하겠습니다.

![readme/Untitled%2030.png](readme/Untitled%2030.png)

서비스에서 **GameLift**를 선택합니다.

작업에서 **모든 GameLift 작업** 을 체크합니다. 

리소스에서 **모든 리소스**를 선택합니다.

![readme/IAM-003.png](readme/IAM-003.png)

![readme/Untitled%2031.png](readme/Untitled%2031.png)

![readme/Untitled%2032.png](readme/Untitled%2032.png)

### Step 5.2. Lambda가 사용할 역할 만들기

프로젝트에서 사용할 Lambda가 2가지 형태가 있습니다. GameLift와 Network하는 Lambda 함수와 RDS 서비스를 사용하는 Lambda 함수가 있습니다. 2가지 Lambda 함수에서 사용할 역할을 나눌 수 있으나 이번 프로젝트에서는 통합 역할을 만들겠습니다.

![readme/Untitled%2033.png](readme/Untitled%2033.png)

![readme/Untitled%2034.png](readme/Untitled%2034.png)

아래의 정책들을 찾아서 체크 한 후 **다음:태그**를 누릅니다.

AmazonRDSDataFullAccess

AWSLambdaSQSQueueExecutionRole

AWSLambdaBasicExecutionRole

AWSLambdaVPCAccessExecutionRole

GameLiftFullAccess

![readme/IAM-008.png](readme/IAM-008.png)

태그를 설정하고 **다음:검토**를 누릅니다.

![readme/IAM-009.png](readme/IAM-009.png)

역할 이름을 설정 후 **역할 만들기**를 누릅니다.

![readme/Untitled%2035.png](readme/Untitled%2035.png)

### Step 5.3. Fleet 인스턴스에서 사용할 역할 만들기

![readme/IAM-010.png](readme/IAM-010.png)

![readme/IAM-011.png](readme/IAM-011.png)

![readme/IAM-012.png](readme/IAM-012.png)

![readme/IAM-013.png](readme/IAM-013.png)

![readme/IAM-014.png](readme/IAM-014.png)

![readme/IAM-015.png](readme/IAM-015.png)

![readme/IAM-016.png](readme/IAM-016.png)

생성된 역할 mzc-game-gamelift-fleet-role의 **역할 ARN** 을 메모합니다.

## Step 6. RDS 생성

### Step 6.1. RDS 서브넷 그룹 생성

**서브넷 그룹**을 생성합니다.

![readme/RDS-001.png](readme/RDS-001.png)

아래와 같이 설정 후 생성을 누릅니다.

![readme/Untitled%2036.png](readme/Untitled%2036.png)

![readme/Untitled%2037.png](readme/Untitled%2037.png)

### Step 6.2. RDS 생성

아래에 없는 수정 내용은 전부 default 로 진행하였습니다.

![readme/Untitled%2038.png](readme/Untitled%2038.png)

![readme/RDS-002.png](readme/RDS-002.png)

![readme/Untitled%2039.png](readme/Untitled%2039.png)

DB 마스터 암호는 **Admin1!!** 로 설정했습니다.

![readme/Untitled%2040.png](readme/Untitled%2040.png)

![readme/Untitled%2041.png](readme/Untitled%2041.png)

**스토리지 자동 조정을 비활성화** 합니다.

![readme/Untitled%2042.png](readme/Untitled%2042.png)

![readme/RDS-006.png](readme/RDS-006.png)

![readme/Untitled%2043.png](readme/Untitled%2043.png)

![readme/Untitled%2044.png](readme/Untitled%2044.png)

생성이 완료되면 **엔드포인트**를 확인합니다. 

![readme/Untitled%2045.png](readme/Untitled%2045.png)

### Step 6.3. Cloud9 생성

DB, Table 생성을 Cloud9 위에서 진행하겠습니다. 

![readme/Untitled%2046.png](readme/Untitled%2046.png)

![readme/Untitled%2047.png](readme/Untitled%2047.png)

![readme/Cloud9-003.png](readme/Cloud9-003.png)

**Create environment** 를 눌러 생성합니다.

### Step 6.4. Database, Table 생성

생성된 Cloud9에서 DB 엔드포인트로 **DB에 접속**합니다.

```bash
mysql -h mzc-game-rds.cqoanobel0pg.ap-northeast-2.rds.amazonaws.com -u admin -p
```

![readme/DBJob-001.png](readme/DBJob-001.png)

Database를 생성합니다.

```sql
create database accounts;
```

![readme/DBJob-002.png](readme/DBJob-002.png)

서버 프로젝트에 있는 테이블 생성문을 복사해서 테이블을 가져옵니다.  

아래 코드를 복사하셔도 됩니다. 

붙여 넣어 테이블을 생성합니다.

(ctc-game-server\DBSchema\accounts\V1__create_table_users.sql)

```sql
use accounts;
CREATE TABLE `users` (
	`userid` INT(11) NOT NULL AUTO_INCREMENT,
	`nickname` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	`win` INT(11) NULL DEFAULT NULL,
	`loss` INT(11) NULL DEFAULT NULL,
	`draw` INT(11) NULL DEFAULT NULL,
	`point` INT(11) NULL DEFAULT NULL,
	PRIMARY KEY (`userid`) USING BTREE,
	UNIQUE INDEX `UX_users_nickname` (`nickname`) USING BTREE
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;
```

![readme/DBJob-003.png](readme/DBJob-003.png)

## Step 7. Lambda 함수 생성 (AccountJoin, Login, BattleResult)

### Step 7.1. AccountJoin Lambda 함수 업로드

![readme/LambdaPublish-001.png](readme/LambdaPublish-001.png)

![readme/LambdaPublish-002.png](readme/LambdaPublish-002.png)

![readme/Untitled%2048.png](readme/Untitled%2048.png)

![readme/LambdaPublish-004.png](readme/LambdaPublish-004.png)

![readme/LambdaPublish-005.png](readme/LambdaPublish-005.png)

### Step 7.2. Login Lambda 함수 업로드

![readme/Untitled%2049.png](readme/Untitled%2049.png)

![readme/Untitled%2050.png](readme/Untitled%2050.png)

![readme/LambdaPublicshLogin-003.png](readme/LambdaPublicshLogin-003.png)

![readme/LambdaPublicshLogin-004.png](readme/LambdaPublicshLogin-004.png)

![readme/Untitled%2051.png](readme/Untitled%2051.png)

### Step 7.3. API Gateway, AccountJoin, Login 리소스 생성

![readme/Untitled%2052.png](readme/Untitled%2052.png)

![readme/Untitled%2053.png](readme/Untitled%2053.png)

**AccountJoin 생성**

![readme/Untitled%2054.png](readme/Untitled%2054.png)

![readme/Untitled%2055.png](readme/Untitled%2055.png)

![readme/APIGateway-008.png](readme/APIGateway-008.png)

![readme/Untitled%2056.png](readme/Untitled%2056.png)

![readme/Untitled%2057.png](readme/Untitled%2057.png)

![readme/Untitled%2058.png](readme/Untitled%2058.png)

![readme/Untitled%2059.png](readme/Untitled%2059.png)

![readme/APIGateway-013.png](readme/APIGateway-013.png)

**URL 확인**

![readme/Untitled%2060.png](readme/Untitled%2060.png)

같은 방법으로 Login도 생성합니다.

**리소스 생성**
리소스 이름 : Login
리소스 경로 : Login
API Gateway CORS 활성화 체크
**메서드 생성**
ANY
Lambda 함수 : Login

**API 배포**

배포 스테이지 : test

![readme/Untitled%2061.png](readme/Untitled%2061.png)

### Step 7.4. BattleResult 람다 함수 업로드

Function Name : BattleResult

Assembly : BattleResult

Type : BattleResult.Function

Method : FunctionHandler

Role Name, VPC Subnets, Security Groups : AccountLogin과 똑같이 설정

![readme/BattleResult-001.png](readme/BattleResult-001.png)

### Step 7.5. SQS 생성

![readme/Untitled%2062.png](readme/Untitled%2062.png)

![readme/Untitled%2063.png](readme/Untitled%2063.png)

나머지는 기본 값으로 **대기열 생성** 합니다.

![readme/Untitled%2064.png](readme/Untitled%2064.png)

![readme/Untitled%2065.png](readme/Untitled%2065.png)

SQS **URL** 을 확인합니다.

![readme/SQS-005.png](readme/SQS-005.png)

## Step 8. 서버 배포

### Step 8.1. GameLift 에 서버 배포를 위한 사용자 변수 설정

GameLift에 빌드를 배포하는 스크립트에서 사용할 사용자 변수를 설정 합니다.

(Windows 기준)

**고급 시스템 설정 보기 > 환경 변수 > 새로 만들기(사용자 변수)**

![readme/Environment-001.png](readme/Environment-001.png)

![readme/Environment-002.png](readme/Environment-002.png)

### Step 8.2. Code 수정 사항

서버 배포 전 하드 코딩 된 부분을 수정해야 합니다.

**\ctc-game-server\BattleServer\Game\Room\Room.cs**

위 경로의 파일에서 **BattleResultSendToSQS** 함수에서 수정합니다. 

```csharp
public void BattleResultSendToSQS(PlayerType winPlayer)
{
    var sqsConfig = new AmazonSQSConfig();
    sqsConfig.ServiceURL = "https://sqs.ap-northeast-2.amazonaws.com/831150897155/mzc-game-sqs";
    sqsConfig.RegionEndpoint = RegionEndpoint.APNortheast2;
    var credential = new BasicAWSCredentials("AccessKey", "SecretKey");
    var sqsClient = new AmazonSQSClient(credential, sqsConfig);

    SQSClientSendMessage(sqsClient, PlayerType.Player1, winPlayer);
    SQSClientSendMessage(sqsClient, PlayerType.Player2, winPlayer);
}
```

아래 부분을 SQS URL과 자신의 AccessKey, SecretKey로 수정해야합니다.

![readme/Untitled%2066.png](readme/Untitled%2066.png)

**SQSClientSendMessage** 함수에서 수정합니다. 

```csharp
private void SQSClientSendMessage(AmazonSQSClient sqsClient, PlayerType player, PlayerType winPlayer)
{
    sqsClient.SendMessageAsync(new SendMessageRequest(
        "https://sqs.ap-northeast-2.amazonaws.com/831150897155/mzc-game-sqs",
        JsonConvert.SerializeObject(new CommonProtocol.SqsBattleResult
        {
            UserId = _players[player].Session.UserId,
            WinType = GetWinType(player, winPlayer)
        })));
}
```

아래 부분에서 SQS URL을 수정합니다.

![readme/Code-002.png](readme/Code-002.png)

URL과 Credentials 참조 부분은  다른 방식(Config, 환경 변수 참조 등)으로 개선 해야 합니다.

### Step 8.3. GameLift 에 업로드할 서버 빌드

빌드를 진행합니다.

![readme/Untitled%2067.png](readme/Untitled%2067.png)

### Step 8.4. GameLift에 업로드할 대상 파일들이 있는 디렉토리로 의존성 파일 복사

아래 경로 2개의 파일을

**\ctc-game-server\BattleServer\Dependency\install.sh**

**\ctc-game-server\BattleServer\Dependency\NDP472-KB4054530-x86-x64-AllOS-ENU.exe**

아래의 경로에 복사합니다.

**\ctc-game-server\BattleServer\bin\Debug**

### Step 8.5. GameLift에 서버 바이너리, 의존성 파일 업로드

GameLift에 빌드를 업로드 합니다.

**\ctc-game-server\scripts\GameLift\01-GameLift-UploadBuild.sh 를** 실행합니다.

![readme/Untitled%2068.png](readme/Untitled%2068.png)

실행 후 나온 **Build ID** 를 메모합니다.

### Step 8.6. 플릿 생성

플릿 생성 스크립트를 수정합니다.

(**\ctc-game-server\scripts\GameLift\02-GameLift-CreateFleet.sh)**

**Build ID**를 수정합니다.

위에서 생성한 역할 mzc-game-gamelift-fleet-role의 **역할 ARN** 을 수정합니다.

![readme/ServerDeploy-003.png](readme/ServerDeploy-003.png)

**\ctc-game-server\scripts\GameLift\02-GameLift-CreateFleet.sh** 를 실행합니다.

GameLift 대시보드로 이동하면 업로드한 빌드와 생성한 플릿이 보입니다.

![readme/Untitled%2069.png](readme/Untitled%2069.png)

플릿 생성 완료까지 약 20분이 소요됩니다. 

플릿 생성과정에서 GameLift 와 통신하는 필수 통합 코드가 있습니다. GameLift는 플릿 인스턴스의 서버 프로세스와 테스트 통신을 통해서 통합 코드를 잘 구현 했는지 검증합니다. 해당 내용을 통과하지 못하면 활성(Activate) 상태로 가지 못합니다. 

정상적으로 작동하는 코드임에도 **활성** 상태로 가지 못할 때가 있습니다. 

만약 **활성화하는 중(Activating)** 상태가 오래 지속되면 다시 플릿 생성을 합니다.

플릿이 정상적으로 활성화 되면 아래와 같이 확인할 수 있습니다.

이벤트 탭에서 플릿 생성 과정을 확인할 수 있습니다.

**플릿 ID**를 메모합니다.

![readme/ServerDeploy-005.png](readme/ServerDeploy-005.png)

이제 플릿과 통신할 수 있는 상태가 되었습니다.

## Step 9. Lambda 함수 생성 (CreateGame, CreatePlayer, SearchGame)

해당 함수들은 플릿 ID가 필요하므로 플릿 생성 시도 후 ID가 나오면 생성을 진행 할 수 있습니다.

### Step 9.1. CreateGame, CreatePlayer, SearchGame, Lambda 함수 업로드

CreateGame 람다 함수를 업로드 합니다.

![readme/Untitled%2070.png](readme/Untitled%2070.png)

![readme/ForGameLiftFuntcionUpload-002.png](readme/ForGameLiftFuntcionUpload-002.png)

![readme/Untitled%2071.png](readme/Untitled%2071.png)

![readme/Untitled%2072.png](readme/Untitled%2072.png)

![readme/ForGameLiftFuntcionUpload-005.png](readme/ForGameLiftFuntcionUpload-005.png)

CreatePlayer, SearchGame 람다 함수도 같은 방식으로 업로드 합니다.

![readme/Untitled%2073.png](readme/Untitled%2073.png)

**CreatePlayer**

Function Name : CreateGame

Assembly : CreatePlayer

Type : CreatePlayer.Function

Method : FunctionHandler

Role Name, VPC Subnets, Security Groups : CreateGame과 똑같이 설정

Variable : GAMELIFT_FLEET_ID

Value :  플릿 ID (fleet-0f9b2bb2-53bc-424a-8682-0xxxxxxxxxxxx)

**SearchGame**

Function Name : SearchGame

Assembly : SearchGame

Type : SearchGame.Function

Method : FunctionHandler

Role Name, VPC Subnets, Security Groups : CreateGame과 똑같이 설정

Variable : GAMELIFT_FLEET_ID

Value :  플릿 ID (fleet-0f9b2bb2-53bc-424a-8682-0xxxxxxxxxxxx)

### Step 9.2. API Gateway, CreateGame, CreatePlayer, SearchGame 리소스 생성

위에서 AccountJoin, Login 생성했던것 과 같은 방법으로 CreateGame, CreatePlayer, SearchGame 리소스를 생성합니다.

**리소스 생성**
리소스 이름 : CreateGame
리소스 경로 : CreateGame
API Gateway CORS 활성화 체크

**메서드 생성**
ANY
Lambda 함수 : CreateGame

**API 배포**

배포 스테이지 : test

**리소스 생성**
리소스 이름 : CreatePlayer
리소스 경로 : CreatePlayer
API Gateway CORS 활성화 체크

**메서드 생성**
ANY
Lambda 함수 : CreatePlayer

**API 배포**

배포 스테이지 : test

## Step 10. 클라이언트

### Step 10.1. Client Config 수정

경로 : **\ctc-game-client\Assets\Resources\Dev\Connect.json**

Address 부분을 API Gateway URL 확인한 경로로 수정합니다.

```json
{
  "GameServer": {
    "Address": "https://a9j3pkj9j9.execute-api.ap-northeast-2.amazonaws.com/test/"
  }
}
```

### Step 10.2. Client Build

![readme/ClientBuild-001.png](readme/ClientBuild-001.png)

![readme/ClientBuild-002.png](readme/ClientBuild-002.png)

Build 바이너리와 종속 파일에 생성될 폴더를 생성 후 상위 경로에서 폴더 선택을 누릅니다. 

![readme/Untitled%2074.png](readme/Untitled%2074.png)

## Step 11. Play

![readme/Untitled%2075.png](readme/Untitled%2075.png)

2개의 클라이언트를 실행하고 **이름을 입력**하고 **가입**합니다. API 호출 시 cold start delay가 있습니다.

한쪽 클라이언트에서는 **방 만들기**를 하고

다른 쪽 클라이언트에서는 **방 검색**을 합니다.

검색된 이름을 클릭하면 게임을 시작합니다.

상단에 있는 초가 0이 되었을 때 최근에 눌렀던 가위, 바위, 보를 가지고 상대방과 비교하여 승패 판정을 하게 됩니다. 

5 라운드가 진행되며 3선 승을 하게 되면 게임이 종료됩니다.

![readme/Untitled%2076.png](readme/Untitled%2076.png)

게임이 끝나고 DB를 조회 해 보면 승리, 패배, 무승부 카운트 및 점수 값이 반영되어 있습니다.

![readme/Play-003.png](readme/Play-003.png)

## Step 12. Next

### Step 12.1. Queue, Alias, Rule Set

### Step 12.2. IoC

### Step 12.3. CI/CD

### Step 12.4. 하드 코딩 부분 수정

하드 코딩된 부분 수정 후 CI/CD에 통합

### Step 12.5. 로그 수집

![readme/FleetLog-001.png](readme/FleetLog-001.png)

Game Server 의 각 Fleet 에서 Kinesis Data Firehose 라는 서비스를 이용해서 가장 쉽고 안정적으로 S3 로 로드할 수 있습니다.

Kinesis Data Firehose 를 사용하신다면 실시간으로 게임 서버에서 발생하시는 로그를 별도의 인프라 관리의 필요없이 S3 로 준실시간으로 수집할 수 있는 파이프라인을 제공합니다.

말씀드린 것처럼 Kinesis Data Firehose 는 데이터를 전송하는 버퍼 역할을 수행하며, 1-128MB 단위로 값을 지정해서 버퍼 크기 기준 또는 60-900초 간격 사이에서 값을 지정하여 버퍼 시간 기준으로 S3 에 스트리밍 데이터를 전달합니다.

따라서 가장 실시간성으로는 데이터 쌓이는 시간이 1분, 데이터 쌓이는 양이 1MB 중 더 빠른 시간안에 데이터를 전송하실 수 있다고 생각해주시면 됩니다.

([https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/what-is-this-service.html](https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/what-is-this-service.html))

([https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/basic-deliver.html](https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/basic-deliver.html))

S3 에 적재하신 이후에는 S3 를 중심 Data Lake 로 사용하시거나, 목표하시는 바대로 빅쿼리로 옮기셔서 중앙 관리하실 수 있는 방안이 있으실 것으로 보입니다. 

## Step 13. ETC

### Step 13.1. Amazon GameLift Primer Script

[Amazon GameLift Primer](https://www.aws.training/Details/eLearning?id=66534) 과정은 GameLift를 이해하는데 많은 도움이 됩니다. 

해당 과정의 스크립트를 공유합니다.

- Amazon GameLift Primer(개요)
    
    Amazon GameLift Primer를 시작하겠습니다. Amazon에서 기술 교육 과정 개발자로 일하고 있으며  이 과정의 진행을 맡은 Tom Stern입니다.
    
    이 과정에서는 Amazon GameLift, 이 서비스 주요 기능과 게임에서 서비스를 사용하는 다양한 방법에 대해 알아봅니다.
    
    GameLift는 게임 호스팅을 위한 관리형 서비스입니다. 대부분은 게임 소프트웨어를 실행할 EC2 인스턴스를 제공하는 것처럼 들리지만 GameLift는 프로그램을 실행하는 것 이상의 기능을 수행합니다. GameLift는 모든 종류의 세션 기반 게임에 사용할 수 있도록 설계된 전용 게임 서버 호스팅을 제공합니다. 정교한 제어 기능으로 게임 호스트 인스턴스의 플릿을 관리하므로 비용을 제어하면서 인프라 리소스를 확장할 수 있습니다. GameLift는 게임 세션을 관리합니다. 플레이어 지연속도를 고려해 리전에 게임을 배치합니다..
    
    GameLift는 멀티플레이어 게임의 플레이어 요청을 관리하고 정교한 알고리즘을 사용하여 모든 요구 사항을 충족하는 최적의 매치를 찾습니다. GameLift는 확장성, 유연성 및 비용 효율성이 뛰어납니다.
    
    이 과정의 기본적인 목표는 GameLift를 빠르게 배우는 것입니다.
    
- 게임 호스팅 요구 사항
    
    GameLift에서 제공하는 기능은 게임용으로 특별히 설계되었습니다. 이러한 기능은 매우 기술적일 수 있으므로 먼저 컨텍스트 및 배경 정보를 배우게 됩니다.
    
    새 게임을 작업 중인 게임 스튜디오의 예를 생각해 보십시오.
    
    이 스튜디오에서는 멀티플레이어 게임을 개발 중이었습니다.
    
    게임이 준비되었다고 생각되었고
    
    테스트를 실행했습니다.
    
    팀원들은 결과를 기다리면서 사람들이 로컬 네트워크에서 게임을 플레이하는 것을 지켜보고 있습니다. 통신 지연은 아주 짧습니다. 플레이어 의견은…
    
    …모든 플레이어가 좋아합니다!
    
    좋은 소식입니다. 이제 게임을 한 수준 위로 끌어올려야 합니다.
    
    게임을 성장시키기 위한 기본적인 요구 사항은 무엇일까요?
    
    확장 가능한 솔루션입니다. 많은 수의 플레이어를 지원하고 싶다면 더 많은 게임 서버를 호스팅해야 합니다. 또한 용량을 초과하여 실행하는 일은 없어야 합니다. 비용 효율성도 중요하므로 유지 관리가 간편한 솔루션이 필요합니다. 버그를 수정하기 위한 패치나 게임에 추가된 새로운 기능이 있는지 여부에 관계없이 새로운 플레이어를 최신 버전의 게임으로 연결하고 기존 플레이어는 실행 중인 게임 세션을 완료할 수 있어야 합니다. 게임 서비스가 계속 실행되고 사용 가능한 상태로 유지되는 동안 게임을 업그레이드할 수 있어야 합니다. 따라서 안정적인 솔루션이 필요합니다. 게임 서버가 충돌하거나 연결할 수 없게 되더라도 전체 게임 서비스가 다운되는 일은 없어야 합니다. 플레이어는 다시 로그인하여 계속 플레이할 수 있어야 합니다. 마지막으로, 게임에 액세스할 수 있는 사용자만 액세스할 수 있도록 시스템을 안전하게 보호해야 합니다. 멀티 플레이어 게임에 필요한 요소는 무엇일까요?
    
    먼저, 매치메이킹이 필요합니다.
    
    멀티플레이어 게임은 플레이하려는 플레이어에게 제공할 수 있는 호스팅 리소스를 찾아 실행 가능한 게임을 만듭니다.
    
    매치메이킹은 규칙 세트를 적용하여 적절한 매치와 허용 가능한 매치, 허용 가능한 매치를 결정하기 전에 적절한 매치를 찾는 데 소요되는 시간을 정의합니다.
    
    플레이어 지연과 같은 특성이나 힘, 속도 및 스킬과 같은 플레이어 속성을 설명하기 위한 규칙을 개발할 수 있습니다.
    
    마지막으로, 특정 수의 플레이어가 필요하도록 게임을 설정하거나 특정 포지션을 가진 팀을 정의할 수 있습니다. 자체 매치메이킹 소프트웨어를 사용하거나 GameLift FlexMatch와 같이 서비스로 제공되는 매치메이킹을 사용할 수 있습니다. 어느 경우든 매치메이킹은 일반적인 게임 호스팅 요구 사항입니다.
    
    특정 수의 팀 또는 특정 플레이어 재능을 가진 팀에 플레이어를 배정해야 할 수도 있습니다. 동일한 플레이어로 팀을 구성할 수 없도록 설정해야 할 수 있습니다. 예를 들어 스포츠 팀의 경우 다른 플레이어와 다른 기술을 가진 골키퍼가 필요할 수 있습니다. 서로 다른 팀 역할 또는 포지션이 자격을 갖춘 플레이어로 채워지도록 하려면 어떻게 해야 할까요? 팀 및 팀 역할을 정의하고 플레이어를 팀에 배정하는 기능은 게임 호스팅의 일반적인 요구 사항 중 하나입니다.
    
    모든 플레이어가 로컬 영역 네트워크에서 플레이할 때는 지연이 거의 없었습니다.
    
    게임을 확장하고 글로벌화하면 플레이어 지연이 게임플레이에 미치는 영향을 고려해야 합니다.
    
    적은 지연시간으로 인해 플레이어 참여도와 리텐션을 향상시킬 수 있습니다.
    
    지리적으로 플레이이와 가까운 게임 서버를 제공하는 것도 게임 호스팅의 일반적인 요구 사항입니다.
    
    매치를 찾은 후에는 게임 세션을 어딘가에서 시작해야 합니다.
    
    기본적으로 플레이어 참여를 유지하면서 게임을 성장시키는 것이 좋습니다. 플레이어를 유치하고 유지하는 데 도움이 되는 훌륭한 게임 경험을 만들어야 합니다.
    
    게임 호스팅 솔루션에는 비즈니스 요구 사항도 있습니다. 비즈니스를 운영하려면 여러 조직에 적절한 정보가 필요합니다. 요구 사항 중 하나는 솔루션이 적절한 비즈니스 의사 결정, 적절한 게임 설계 의사 결정 및 적절한 운영 의사 결정을 내리는 데 필요한 데이터를 제공해야 한다는 것입니다. 또 다른 요구 사항은 가격 투명성입니다. 비즈니스에 대한 건전한 재무 결정을 내리기 위해서는 가격 책정에 대한 명확하고 이해하기 쉬운 정보가 필요합니다. 즉, 리소스 사용률을 스마트하게 예측하고 사용된 리소스의 레코드를 분석하는 기능이 있어야 합니다.
    
    다행히 Amazon GameLift를 사용하면 이 모든 요구 사항을 훨씬 더 쉽게 충족할 수 있습니다.
    
- Amazon GameLift의 주요 이점
    
    GameLift는 게임 서비스에 유용할 수 있는 많은 기능을 제공합니다. 다음 여러 단원에서는 GameLift가 제공하는 서비스의 유연성과 범위에 대해 배우게 됩니다. 글로벌 확장성과 필요에 따라 탄력적으로 인프라를 확장할 수 있는 기능은 게임 스튜디오에서 꾸준히 유용하게 사용되는 두 가지 기능입니다.
    
    GameLift의 가장 유명한 두 가지 이점이 바로 “온디맨드 게임 세션”과 “글로벌 배포”입니다. 온디맨드 게임 세션은 변화하는 플레이어 수요에 따라 확장하고 축소할 수 있는 확장 가능한 인프라를 통해 달성됩니다. 즉, 항상 충분한 게임 세션을 사용할 수 있고 장비에 대한 자본 투자 대신 필요한 만큼만 비용을 지불할 수 있습니다. 글로벌 배포는 여러 리전에 GameLift 솔루션을 배포하여 달성됩니다. GameLift에서는 할인된 요금으로 제공되는 스팟 인스턴스를 사용하여 글로벌 배포의 비용 효율성을 높일 수 있습니다. 가격은 시장 입찰에 의해 설정됩니다. GameLift는 게임 서버를 플레이어 근처에 배치하고 플레이어를 근처의 게임 서버로 연결함으로써 최상의 게임을 제공합니다.
    
    용량을 추측할 필요가 없습니다. GameLift의 기능은 플레이어 수요에 따라 용량을 관리합니다. 용량을 훨씬 빠르게 확장할 수 있습니다. 몇 분 안에 용량을 켜고 끌 수 있습니다. 몇 주가 소요되는 온프레미스 데이터 센터 또는 코로케이션 솔루션과 다릅니다. 또 다른 이점은 사전 하드웨어 프로비저닝 및 시작 비용을 피할 수 있다는 것입니다.
    
    플레이어 문제도 줄어듭니다. 즉, 프로비저닝 부족으로 인한 플레이어 불만이 발생하지 않습니다.. 지연이 짧기 때문에 플레이어 경험이 향상됩니다.
    
    수익화와 비용이 같은 수준으로 증가합니다.
    
- GameLift 이해
    
    Amazon GameLift에서는 세 가지 주요 개념을 이해해야 합니다. GameLift는 4개의 개별적이지만 상호 의존적인 시스템으로 구성됩니다. GameLift는 여러 수준의 추상화에서 사용될 수 있습니다. 또한 GameLift와 상호 작용하는 코드를 세 가지 개별 협력 프로그램으로 구현할 수 있습니다.
    
    Amazon GameLift는 서비스인 동시에 애플리케이션 프레임워크입니다.
    
    서비스는 유틸리티와 같습니다. 수도시설을 생각해보십시오. 물은 수도시설을 통해 주택으로 공급되며 수도꼭지에서 나옵니다. 선택할 수 있는 옵션은 없습니다. 소금물이나 담수 또는 탄산수를 선택할 수 없습니다. 그냥 물 뿐입니다. 또한 사용자는 이 서비스에 작업 방법을 지시할 수 없습니다. 목요일에는 서비스를 중지하거나 토요일에는 수압을 높이라는 등의 요청을 할 수 없습니다.
    
    에게 작업 방법을 지시할 수 있습니다. “바닐라 세 펌프 대신 한 펌프만 넣고 젓지 말고 흔들어 주세요”라고 말할 수 있습니다. 여기에는 몇 가지 기본값이 있습니다. 단순히 “플레인 라지 커피”를 주문할 수 있습니다. 그러나 서비스가 수행되는 프로세스에 큰 영향을 미침으로써 결과에 막대한 영향을 미칠 수 있습니다.
    
    Amazon GameLift는 일반적으로 게임 호스팅 서비스로 설명되지만 일부는 유틸리티처럼 작동하고 다른 일부는 프레임워크처럼 작동합니다. 일부에는 기본값이 없으며 지침과 서비스를 수행하는 방법을 알려주지 않는 한 작동하지 않습니다.
    
    GameLift의 기능, 사용 방법 및 게임 백엔드 솔루션에 제공하는 이점을 이해하려면 GameLift의 여러 부분을 탐색하고 유틸리티와 같은 부분, 사용자 지정이 가능한 부분 및 사용자의 지시에 따르는 부분을 이해하는 것이 중요합니다.
    
    Amazon GameLift는 멀티 플레이어 게임을 호스팅하는 서비스입니다. GameLift는 게임 세션을 시작하고 게임 백엔드에서 여러 플레이어를 게임 세션에 연결하는 데 사용됩니다. GameLift의 작동 방식을 이해하려면 몇 가지 주요 구성 요소와 해당 구성 요소를 사용하는 시스템에 익숙해져야 합니다. 이 단원에서는 GameLift 시스템과 주요 구성 요소에 대해 배웁니다. GameLift는 싱글플레이어 게임에 유용할 수 있습니다. 그러나 이 서비스는 멀티플레이어용으로 설계되었습니다.
    
    GameLift는 4개의 개별적이지만 상호 의존적인 시스템으로 구성됩니다. 각 계층은 그 아래의 계층을 사용하며 포함합니다.
    
    매치메이킹은 게임을 플레이하려는 플레이어와 게임을 플레이할 수 있는 게임 세션을 연결하는 것입니다.
    
    게임 세션 배치는 게임 세션을 호스팅할 위치를 결정합니다.
    
    세션 관리는 게임을 시작하고 플레이어의 게임 참여를 용이하게 합니다.
    
    인프라 관리는 게임 서버에 대한 탄력적인 제어를 제공합니다.
    
    상위 스택으로 갈수록 GameLift는 자유도와 사용자 지정 가능성이 증가하는 프레임워크처럼 작동합니다. GameLift는 기존의 다른 서비스와 달리 사용자 지정할 수 있는 항목이 매우 많고 아주 유연합니다. GameLift에서는 많은 작업이 가능하지만 이러한 작업을 수행하는 방법과 시기는 코드에 따라 결정됩니다. GameLift에 작업 방법을 지시하는 코드를 게임 서비스 로직이라고 합니다. GameLift를 사용하는 방법을 배우는 것은 GameLift가 게임에 필요한 방식으로 동작하도록 게임 서비스 로직을 설계하고 구현하는 방법을 배우는 것입니다.
    
    인프라 관리 시스템은 컴퓨팅, 스토리지 및 네트워킹 리소스를 담당합니다. GameLift는 EC2를 기반으로 구축됩니다. 이 시스템은 여러 리전에서 EC2 인스턴스 플릿을 관리합니다. 또한 각 EC2 인스턴스에 설치된 게임 소프트웨어와 소프트웨어에서 구축된 게임 서버 프로세스를 담당합니다.
    
    게임 세션은 인프라 관리와 다른 시스템 간의 경계에 있습니다. 프로세스 로드 및 시작 또는 프로세스 종료 및 교체 등 프로세스와 관련된 모든 작업은 인프라 관리에 속합니다. 게임 세션 자체와 관련이 있는 모든 작업(예: 플레이할 게임의 준비 상태 선언, 플레이어 슬롯 예약 또는 게임플레이)은 인프라 관리 외부에서 발생합니다. 인프라 관리는 모든 GameLift 시스템의 유틸리티처럼 동작합니다. 리소스를 요청하면 사전 구성된 한도 내에서 자동으로 리소스가 제공됩니다.
    
    매치메이킹에는 GameLift에 로드되는 규칙이 포함됩니다. 규칙은 시스템의 의사 결정 방식을 결정합니다. 따라서 서비스 운영은 사용자가 결정합니다. 그런 다음에는 게임 백엔드와 게임 서버의 코드가 게임 서비스 로직을 구현합니다. 이 로직은 결정이 내려지는 시기와 그 결과로 수행할 작업을 결정합니다. GameLift의 매치메이킹 서비스를 FlexMatch라고 합니다. GameLift에는 하나 이상의 매치메이커가 있을 수 있습니다. 매치메이커는 게임에 참여할 수 있는 플레이어 수, 팀 수, 게임에서 사용할 게임 맵, 게임에서 사용할 모드 등 요청되는 게임의 종류와 관련된 플레이어 정보 및 세부 사항을 포함하는 정의 풀을 담당합니다. 매치메이커에는 유효한 게임의 구성 요소를 정의하는 규칙 세트가 있습니다. 일치가 발생하려면 요청의 모든 기준과 규칙의 모든 기준이 충족되어야 합니다. 매치메이커는 기본적으로 요청 풀과 사용 가능한 인프라를 반복하여 유효한 게임 구성을 찾습니다. 모든 요구 사항을 충족하는 리소스를 찾으면 매치를 선언합니다.
    
    GameLift 아키텍처는 각 계층이 하위 계층을 래핑하고 구동합니다. 매치메이킹은 플레이어 요청과 리소스 요구 사항은 알고 있지만 게임을 호스팅하는 데 사용할 수 있는 리소스로 EC2 인스턴스를 찾는 방법은 알지 못합니다. 이 지원을 위해 서비스는 다음 계층인 게임 세션 배치로 이동합니다.
    
    게임 세션 배치 시스템의 주요 기능은 리전, 리전 내의 플릿, 게임을 호스팅하는 데 사용할 수 있는 리소스가 있는 EC2 인스턴스를 식별하는 것입니다. 게임 세션 배치는 리전에 대한 평균 플레이어 지연을 고려합니다. 기본적으로 플레이어의 게임 소프트웨어는 리전 엔드포인트에 대한 ping을 수행하고 요청에서 지연의 측정값을 보고합니다. 이 측정값은 매치메이킹을 통해 전달되며 결국 게임 세션 배치로 전달됩니다. 게임 세션 배치는 또한 다른 플릿의 비용에 대해 알고 있으므로 사용 가능한 경우 더 낮은 비용 옵션을 사용합니다. 지연 정보를 포함하지 않는 요청을 구성할 수 있으며, 비용이 유사한 인스턴스 플릿을 구성할 수 있습니다. 이 경우 게임 세션 배치는 가용성을 기준으로 의사 결정을 내립니다. 먼저 플릿에 연결할 수 있고 메타데이터에서 사용 가능한 상태로 표시되는지 여부를 고려한 다음 필요한 게임 세션을 호스팅하는 데 사용할 수 있는 게임 서버 프로세스가 있는 EC2 인스턴스가 플릿에 포함되어 있는지 여부를 고려합니다. GameLift 게임 세션 배치 동작을 결정하는 주요 메커니즘은 게임 세션 배치 대기열이라는 객체의 구성에 따라 결정됩니다. 게임 세션 배치 대기열에는 기본 사용 순서에 따라 플릿의 목록이 포함됩니다. 사용자가 정의하는 규칙 세트도 있습니다. 게임 세션 배치 시스템의 지능형 알고리즘은 사용 가능한 정보를 사용하여 게임을 호스팅할 최상의 게임 서버를 찾습니다. 게임 세션 배치는 모든 옵션을 반복하여 게임을 호스트할 장소를 찾습니다. 배치를 즉시 찾을 수 없으면 요청을 대기열로 반환합니다. 요청은 배치가 발견되거나 요청이 최대 대기열 보류 시간을 초과할 때까지 보류되고 주기적으로 다시 방문됩니다.
    
    게임 세션 배치는 게임에 대한 게임 서버를 식별할 수 있습니다. 하지만 게임 서버에서 게임 세션을 시작하거나 게임에서 플레이어 슬롯을 예약할 수는 없습니다. 이러한 작업에는 세션 관리 시스템이 사용됩니다.
    
    세션 관리는 게임을 시작하고 게임 백엔드에서 플레이어를 게임에 연결할 수 있도록 연결 정보를 제공하는 시스템입니다. 그러나 이 동작은 대부분 코드에 의해 결정됩니다. 게임 백엔드에 작성된 코드에 따라 시스템이 게임을 시작하는 방법이 결정됩니다. 또한 게임 서버에 작성된 코드가 게임 중에 시스템이 작동하는 방식을 결정합니다.  서비스를 제어할 때는 API 호출을 사용합니다. 게임 세션이 GameLift에 의해 자발적으로 시작되지 않는다는 것을 생각하면 제어 수준을 이해할 수 있습니다. 게임 세션은 코드가 API를 호출할 때 그 결과로만 시작됩니다.
    
    플레이어 세션은 게임 내 플레이어 슬롯에 대한 요청을 나타내는 GameLift 객체입니다. 플레이어 세션 요청은 매치메이킹 시스템이나 게임 세션 배치 시스템에서 시작되지만 세션 관리 시스템에서 처리되고 해결됩니다.
    
- 게임 서비스 로직
    
    게임 서비스 로직
    
    근본적으로 GameLift의 작동 방식은 코드의 영향을 받습니다.
    
    일부 구성 요소에서는 제한 및 파라미터를 사용하여 GameLift를 구성합니다. 다른 구성 요소에서는 API를 통해 GameLift 함수를 호출합니다. 또 다른 구성 요소에서는 GameLift의 의사 결정에 사용되는 규칙을 생성하고 업로드합니다.
    
    게임 서비스 로직은 코드와 GameLift의 인터페이스 방식을 나타냅니다.
    
    게임 서비스 로직은 게임 백엔드, 게임 서버 및 플레이어의 게임 소프트웨어에 작성되는 코드이며, GameLift의 동작을 요구 사항에 맞게 조정합니다. 이 코드를 작성하려면 코드와 GameLift 시스템 간의 일반적인 상호 작용을 이해해야 합니다. 또한 GameLift 시스템 간의 상호 작용과 API 호출 코드에 포함된 파라미터가 시스템에서 전달되고 처리되는 방식을 이해해야 합니다. 마지막으로 GameLift가 코드에 정보를 반환하는 방법을 이해해야 합니다.
    
    게임 설정은 게임 백엔드에서 실행되는 코드에 의해 제어됩니다. 설명서에서는 게임 백엔드를 클라이언트 또는 클라이언트 서버라고 부르기도 하지만 이 과정에서는 명확성을 위해 항상 게임 백엔드라고 합니다. 이 코드는 AWS Lambda 함수, 컨테이너(예: AWS Fargate) 또는 Elastic Beanstalk에서 실행될 수 있습니다. 그러나 가장 일반적으로 게임 백엔드는 EC2의 웹 서버에서 실행됩니다. 코드는 GameLift Client SDK(AWS SDK의 일부)를 사용하여 GameLift 시스템과 인터페이스합니다. GameLift에서 활용하는 자동화 및 추상화의 양과 사용하는 제어 기능 간의 균형을 선택해야 합니다. 예를 들어 게임을 시작하는 것은 **StartMatchmaking( )**과 같은 API 호출을 사용하여 매치메이킹 시스템과 통신하는 것과 같이 간단한 작업일 수 있습니다. 이 단일 호출로 매치메이킹 아래의 모든 계층에서 작업이 실행됩니다. 하지만 게임 백엔드에 매치메이킹 소프트웨어가 이미 구현되는 있는 또 다른 예를 생각해보십시오.  이 경우에는 **StartGameSessionPlacement**와 같은 API 호출을 사용하여 게임 세션 배치 시스템과 통신하는 방법으로 시작하는 것이 좋을 수 있습니다. 마찬가지로, 이 호출을 수행하면 그 아래의 모든 계층에서 작업이 실행됩니다.
    
    GameLift 활동의 상태는 모든 계층에서 **Describe** API 호출을 통해 확인할 수 있습니다. 이 접근 방식을 사용하면 게임 백엔드의 설계를 GameLift의 작동과 분리할 수 있습니다. 업데이트 빈도는 폴링을 통해 제어됩니다.
    
    GameLift의 추상화 수준을 사용하여 자동화와 제어 간의 균형을 직접 조절할 수 있습니다. 예를 들어 디스크 드라이브가 있습니다. 디스크 드라이브의 파일 시스템에는 GameLift와 비슷한 하위 수준을 포함하는 여러 수준의 추상화가 있습니다.
    
    가장 높은 수준의 추상화에는 전체 디스크와 디스크에 있는 모든 항목(예: “디스크 백업”명령)에 영향을 미치는 명령이 있습니다. 다음 수준에는 폴더 및 디렉터리와 “이동한 폴더”와 같은 명령이 있으며 이는 그 안의 모든 파일에 영향을 줍니다. 다음 수준에는 “열기”, “저장” 및 “닫기”와 같은 개별 파일 명령이 있습니다. 그리고 가장 낮은 수준에는 디스크의 섹터에 직접 이진 데이터를 읽고 쓰는 명령이 있습니다. 디스크와 마찬가지로 GameLift에는 네 가지 추상화 수준의 API 함수가 있습니다.
    
    디스크의 섹터에 직접 이진 데이터를 쓸 일은 없습니다. 파일 I/O가 훨씬 편리하기 때문입니다. 그러나 필요한 경우 이진 읽기 및 쓰기 함수가 있으니 사용할 수 있습니다.
    
    마찬가지로, 개별 플레이어 요청을 관리할 필요도 없습니다. 매치메이킹과 게임 세션 배치가 훨씬 편리합니다. 그러나 필요한 경우 해당하는 API 호출을 사용할 수 있습니다.
    
    한 가지 정보를 알려드리겠습니다. 온라인 참고 자료에서 GameLift API 함수는 알파벳순으로 나열됩니다.  시스템별로 그룹화되거나 추상화 수준으로 태그되어있지 않습니다.**StartGameSessionPlacement( )**와 **CreateGameSession( )**은 관련되어 있습니다. 그러나 **CreateGameSession( )**은 이미 **StartGameSessionPlacemement( )**에서 호출됩니다.**CreateGameSession( )**은 제어를 강화하고 자동화를 줄이려는 경우에만 사용합니다.
    
    게임 백엔드는 게임이 시작되기 전의 설정을 담당합니다.
    
    플레이어의 게임 소프트웨어는 매치 또는 게임 플레이를 위한 요청을 게임 백엔드로 전달합니다. 게임 백엔드는 플레이어를 인증하고 메타데이터를 수집하며 GameLift Client SDK를 통해 GameLift에 플레이어를 보여주는 브로커 역할을 합니다. 게임 백엔드는 게임 설정 프로세스를 전적으로 담당하며 GameLift는 이 활동을 지원하는 서비스를 게임 백엔드에 제공한다고 생각하면 됩니다. 서비스가 아니라 클라이언트가 담당합니다.
    
    게임 서버는 게임플레이를 담당합니다. 또한 게임 설정 중에 게임 백엔드와 파트너 관계를 맺습니다.
    
    게임 서버 코드는 사용자가 작성합니다. 이 코드는 Windows 서버 또는 Linux를 대상으로 합니다. GamLift Server SDK도 포함되어 있습니다.
    
    게임 서버의 목적은 게임플레이 중에 플레이어의 게임 소프트웨어에 게임 시뮬레이션 등의 서비스를 제공하는 것입니다.
    
    그러나 게임 서버는 게임 설정을 지원할 책임도 있습니다. 예를 들어 게임 세션에서 플레이어를 받을 준비가 되었다고 선언하는 작업은 게임 서버 소프트웨어가 담당합니다. 게임 서버 소프트웨어는 API 호출을 통해 이 상태를 GameLift에 알립니다. 플레이어가 게임 세션을 나갈 때 이 빈 슬롯을 추가 플레이어로 채우려는 경우, 즉 채우기 프로세스를 수행하려는 경우 게임 서버 코드에서 API 호출을 사용하여 매치메이킹 시스템을 통해 이 작업을 수행할 수 있습니다.
    
    게임 서버는 GameLift 인프라 관리의 일부인 EC2 인스턴스에서 실행됩니다. 인프라 관리의 한 가지 기능은 게임 서버 프로세스를 시작하고 그 수가 올바른지 확인하는 것입니다. 게임 서버 코드는 실제로 인프라 관리 시스템 내부에서 실행됩니다.
    
    게임 서비스는 주로 2개의 비동기식 프로그램으로 구동됩니다. 게임 서버 소프트웨어와 게임 백엔드 소프트웨어인데 이 두 소프트웨어는 GameLift API 호출을 통해 협업합니다.
    
    이제 GameLift로 구축된 게임 서비스의 기본 구성 요소를 개괄적으로 이해할 수 있습니다.
    
    코드는 게임 서버, 게임 백엔드 혹은 플레이어의 게임 소프트웨어에서 돌아갑니다.
    
    GameLift에 대한 제어는 세 가지 방법이 있습니다. JSON 파일로 업로드된 구성 데이터, GameLift SDK 또는 GameLift 대시보드의 설정을 통해 이루어집니다.
    
    이제 GameLift 게임 서비스 로직의 작동 방식을 설명하기에 충분한 내용이 정의되었으니
    
    일반적인 게임 서비스 로직 흐름을 살펴보겠습니다.
    
    게임 서비스 로직 흐름
    
    이 단원에서는 일반적인 게임 서비스 로직의 흐름을 살펴봅니다.
    
    게임 요구 사항에 맞게 이 흐름을 수정할 수 있습니다. GameLift의 부분과 코드 간의 상호 작용을 이해하는 데 도움이 되는 일반적인 흐름도 이 단원에서 살펴봅니다.
    
    게임플레이어 소프트웨어는 게임 백엔드에 게임 플레이 또는 다른 플레이어와의 매치 플레이를 위한 요청을 보냅니다.
    
    게임 백엔드는 GameLift와 조율하여 매치를 생성합니다. 매치메이킹 시스템은 게임 세션 배치를 사용하여 게임에 가장 적합한 리소스를 찾습니다.
    
    게임을 시작할 리전과 특정 플릿을 결정하는 것입니다. 게임 세션 배치는 게임 세션을 시작할 위치를 결정하지만 시작하지는 않습니다. 게임 시작은 다음 계층의 책임입니다.
    
    게임 세션 배치는 게임 서버에서 게임 세션의 시작을 요청합니다. 이 작업은 세션 관리 시스템에 의해 수행됩니다.
    
    게임 세션 배치는 세션 관리 시스템과 함께 작동하여 실제로 게임을 시작하고 게임에서 플레이어 슬롯을 예약합니다.
    
    게임 서버는 각 시스템을 통해 다양한 작업을 수행하고 상호 작용합니다.
    
    GameLift는 플레이어 예약을 포함한 게임 연결 정보를 게임 백엔드로 반환합니다. 게임 백엔드 코드에 사용되는 추상화 수준에 따라 세션 관리의 응답이 스택의 위로 다시 이동하고 해당하는 수준의 추상화에 의해 게임 연결 정보가 반환됩니다. 이 예에서는 매치메이킹 시스템에 의해 반환됩니다.
    
    게임 백엔드는 플레이어의 게임 소프트웨어에 대한 연결 정보를 제공합니다.
    
    플레이어의 게임 소프트웨어는 게임 세션에 연결하고 예약된 슬롯을 가져옵니다.
    
    이제 게임 서버와 플레이어 게임 소프트웨어 사이에서 게임플레이가 직접 시작됩니다.
    
    게임 세션은 플레이어 예약을 가져왔고 매치메이킹 프로세스가 완료되었음을 GameLift에 알립니다. 게임 서버는 세션 관리에 플레이어가 참여했음을 알립니다. 그러면 슬롯이 RESERVED에서 ACTIVE로 변경됩니다. 이 정보를 세션 관리 시스템에 다시 제공하면 GameLift가 대시보드에 현재 지표 정보를 표시할 수 있습니다.
    
    게임플레이 중에 플레이어가 게임을 나간 경우 게임 서버는 매치메이킹 시스템의 채우기 기능을 사용할 수 있습니다.
    
    채우기는 대체 플레이어를 식별하고 플레이어가 진행 중인 게임에 참가할 수 있도록 합니다. 채우기는 주로 게임 서버 코드에서 구동되지만 초기 매치는 주로 게임 백엔드 코드에서 구동됩니다. 채우기에는 두 가지 종류가 있습니다.
    
    첫 번째는 서버 요청 채우기입니다.
    
    게임 서버가 **StartMatchBackfill( )**을 호출하고 게임의 모든 플레이어에 대한 매치메이킹 데이터를 제공합니다.
    
    그러면 매치메이커에서 매치메이킹 티켓이 생성됩니다. 해당 티켓이 매치에 참여하는 경우 매치메이커는 추가 플레이어를 포함한 **onUpdateGameSession( )** 콜백 함수를 사용하여 서버를 다시 호출합니다. 연결 정보는 매치메이커에 의해 게임 백엔드로 반환되며, 이 정보는 플레이어의 게임 소프트웨어에 IP 주소 및 포트 및 플레이어 세션 토큰으로 전파됩니다.
    
    두 번째 채우기는 자동 채우기입니다.
    
    자동 채우기는 게임이 시작되었지만 모든 플레이어가 참가하지 않은 특별한 경우를 처리하는 효율적인 방법입니다.
    
    자동 채우기는 매치메이킹 구성의 속성중 하나입니다.
    
    자동 채우기가 활성화되면 GameLift가 게임 세션에 열린 슬롯이 없을 때까지 **StartMatchBackfill()** 요청을 반복적으로 생성합니다.
    
    게임에 열린 슬롯이 없으면 자동 채우기가 중지됩니다. 플레이어가 게임을 나가는 경우 서버에서 요청한 채우기를 사용해야 합니다.
    
    Realtime 서버
    
    Realtime 서버는 GameLift에서 제공하는 특수한 종류의 게임 서버의 이름입니다.
    
    지금까지는 이를 게임 서버 소프트웨어라는 일반적인 용어로 지칭했습니다.
    
    사실, 게임 서버에는 두 가지 종류가 있습니다. 지금까지 설명한 게임 서버를 사용자 지정 게임 서버라고 합니다. **사용자 지정 게임 서버**를 사용할 때는 GameLift와의 상호 작용을 포함한 모든 게임 서버 코드를 직접 작성해야 합니다.
    
    게임 서버의 다른 유형으로 **Realtime 게임 서버**가 있습니다. Realtime 게임 서버에는 이미 GameLift와의 필수 상호 작용 대부분을 제공하는 기능이 포함되어 있습니다.
    
    Realtime 서버는 공동 책임 모델을 사용합니다. GameLift가 게임 서버의 일부를 제공하고 사용자가 게임 서버의 또 다른 일부를 스크립트 형태로 제공하는 것입니다. 두 가지 유형의 게임 서버 모두 실제로 실시간으로 작동한다는 점에 유의하십시오.
    
    Realtime 서버는 서버가 실행되는 Node.js 실행 환경을 제공합니다.
    
    Realtime 게임 서버에는 GameLift와의 인터페이스가 이미 작성되어 있기 때문에 개발이 간단합니다.
    
    Realtime 게임 서버에 필요한 단일 Node.js javascript 서버 스크립트가 포함되어 있습니다.
    
    Realtime 게임 서버를 사용하여 개발을 시작하려면 최소 서버 스크립트를 로드하여 패킷 릴레이(메시지 전달)를 수행해야 합니다. 이 예제는 온라인 설명서에서 사용할 수 있습니다.
    
    이 경우 Realtime 게임 서버는 플레이어의 게임 소프트웨어 간에 메시지를 전달하는 릴레이 서비스처럼 작동합니다. 서버 스크립트를 작성하여 추가 게임 서버 기능을 생성할 수 있습니다. 예를 들어 게임 서버에서 시뮬레이션 및 상태 조정을 수행할 수 있습니다.
    
    플레이어 게임 소프트웨어에 포함할 수 있는 별도의 Realtime Servers Client API가 있습니다. 이 API는 Realtime 게임 서버에 이미 구축되어 있는 Realtime 구성 요소와 통신합니다.
    
    Realtime 게임 서버의 이점 중 하나는 플릿 수정 없이 게임 서버에서 스크립트를 전환할 수 있다는 것입니다. 현재 게임 세션은 변경되지 않지만 새로운 게임 세션이 시작되면 Realtime 게임 서버 소프트웨어가 업데이트된 스크립트를 확인하고 대체 스크립트로 전환합니다.
    
    Realtime 서버의 또 다른 이점은 플레이어 게임 소프트웨어와 실시간 게임 서버 간에 전송 계층 보안(TLS)을 활성화하여 암호화된 통신을 쉽게 설정할 수 있다는 것입니다.
    
    Realtime 게임 서버를 사용하려면 플레이어의 게임 소프트웨어를 개발할 때 Realtime Client SDK를 사용합니다.
    
    이렇게 하면 플레이어의 게임 소프트웨어에서 Realtime 게임 서버와 직접 통신할 수 있습니다.
    
    이미 Javascript로 작성된 게임 서버 스크립트(예: 이미 GameSparks를 사용하여 개발된 솔루션)가 있는 경우 Realtime 게임 서버를 사용하는 것이 좋습니다.
    
    이렇게 하면 코드 사용이 비교적 쉬워지고 대규모 환경에서도 잘 작동합니다.
    
    새 게임 서버 소프트웨어를 개발하는 경우 Node.js 실행 환경을 사용하지 않는 한 사용자 지정 게임 서버를 사용하는 것이 좋습니다.
    
    보안 통신
    
    AWS에는 사용자가 보안의 일부를 책임지고 Amazon은 다른 부분에 대한 책임을 지는 공동 보안 모델이 있습니다. 따라서 GameLift에서 이러한 책임이 어떻게 배정되는지 알아야 합니다. 첫 번째로 고려해야 할 영역은 플레이어의 게임 소프트웨어와 게임 세션 간의 보안 통신입니다.
    
    GameLift는 플레이어의 게임 소프트웨어와 게임 세션 간의 전송 계층 보안(TLS)을 지원합니다. 플릿에서 TLS를 활성화하면 플릿을 생성할 때 TLS 인증서가 생성되고 일반적으로 AMI 또는”아미”라고 하는 Amazon Machine Image에 배치됩니다. 이 AMI에서 인스턴스가 생성됩니다. 전체 플릿에 대해 하나의 인증서가 생성됩니다. 전체 플릿에 대해 하나의 인증서가 생성됩니다.
    
    게임 서버에는 인증서를 사용하여 서버 ID를 검증하기 위한 DNS 이름이 있어야 합니다. 플릿 확장에 의해 인스턴스가 생성되면 GameLift가 각 인스턴스에 대한 DNS 항목을 Route53에 생성합니다. 인증서의 DNS 이름은 와일드카드 일치를 사용하여 서버의 FQDN과 일치해야 합니다. 인스턴스가 종료되면 GameLift가 인스턴스에 대한 DNS 항목을 제거합니다. DNS 항목과 인증서를 모두 사용하여 서버를 식별할 수 있습니다.
    
    플레이어의 게임 소프트웨어는 먼저 플릿 서버 인스턴스에 대해 GameLift가 생성한 DNS 이름을 사용하여 TLS를 통해 서버에 연결하려고 시도합니다.
    
    게임 서버 코드는 **GetInstanceCertificate( )**를 호출하여 GameLift가 경로명을 인증서에 반환하도록 합니다. 그러면 코드가 인스턴스에서 인증서를 가져와 보안 연결을 위해 플레이어의 게임 소프트웨어의 요청에 응답하는 데 사용합니다.
    
    플레이어의 게임 소프트웨어는 요청에 대한 응답으로 게임 서버로부터 인증서를 받습니다. 그런 다음 인증서로 게임 서버 ID를 검증하기 시작합니다. 인증서는 인증 기관의 공개 키로 서명됩니다. 여러 인증 기관이 체인에 있을 수 있습니다. 이러한 서명이 유효한 경우 가장 중요한 것은 루트 인증 기관입니다. 플레이어의 게임 소프트웨어에는 루트 인증 기관의 퍼블릭 키 세트가 있습니다. 그 중 하나가 인증서의 루트 인증과 일치하면 인증서가 신뢰되고 게임 서버의 ID가 설정됩니다.
    
    이 시점에서 게임 세션과 플레이어의 게임 소프트웨어는 TLS 구현을 제공하여 트래픽을 암호화하도록 선택할 수 있습니다. Realtime 게임 서버의 경우 이 구현이 기본적으로 제공되지만 사용자 지정 게임 서버는 게임 서버 코드에 따라 다릅니다.
    
    Realtime 서버를 사용하는 게임의 경우 한 단계를 더 제공합니다.
    
    GameLift에서 AWS 리소스 사용
    
    이 단원에서는 게임 서버와 AWS의 다른 리소스 간의 보안 통신에 대한 대안을 살펴보면서 공동 보안 모델에 대한 설명을 계속합니다.
    
    GameLift 서비스 역할은 IAM 권한 정책에 따라 GameLift 플릿 인스턴스에 임시 자격 증명을 부여하여 인스턴스가 AWS 리소스를 사용할 수 있도록 합니다.
    
    플릿을 생성할 때 **CreateFleet( )** 요청에서 **InstanceRoleArn**을 지정할 수 있습니다. 인스턴스 역할 ARN을 설정하면 이 플릿의 인스턴스에서 실행되는 모든 애플리케이션이 설치 스크립트, 서버 프로세스 및 데몬(백그라운드 프로세스) 같은 역할을 수임할 수 있습니다.
    
    AWS Management Console의 IAM 대시보드를 사용하여 역할을 생성하거나 역할의 ARN을 조회할 수 있습니다.
    
    AWS 리소스를 사용해야 하는 경우를 예로 들자면 DynamoDB 데이터베이스에서 플레이어 데이터를 가져와야 할 때입니다. 데이터베이스 리소스에 액세스할 때는 일반적으로 AWS SDK와 **DynamoDB_GetItem( )**과 같은 호출을 사용합니다. 모든 AWS 리소스에는 AWS SDK API를 사용하여 액세스할 수 있습니다.
    
    AWS 계정의 VPC에 있는 EC2 인스턴스를 GameLift 인스턴스와 함께 사용하려는 경우 VPC 간 통신으로 인해 퍼블릭 인터넷으로 통신이 이동합니다. VPC 피어링을 사용하면 트래픽이 AWS 네트워크 내에서 라우팅됩니다.
    
    VPC 피어링 관계를 생성하려면 두 단계를 수행합니다. EC2 인스턴스가 있는 VPC는 GameLift가 담당하므로 첫 번째 단계는 **CreateVpcPeeringAuthorization( )**을 호출하여 GameLift 플릿에 대한 VPC와 VPC 간의 피어 연결을 생성하거나 삭제하기 위한 권한을 요청하는 것입니다.  권한 부여는 **DeleteVpcPeeringAuthorization( )** 호출을 통해 취소되지 않는 한 24시간 동안 유효합니다.  두 번째 단계는 **CreateVpcPeeringConnection( )**을 호출하여 피어링 연결을 설정하는 것입니다.
    
    Amazon GameLift 플릿 관리에 사용하는 계정을 포함하여 액세스 권한이 있는 모든 AWS 계정이 소유한 VPC와 피어링할 수 있습니다. 다른 리전에 있는 VPC와는 피어링할 수 없습니다.
    
- 인프라 관리 시스템
    
    인프라 관리 시스템
    
    GameLift에서 인프라는 게임 호스팅에 필요한 컴퓨팅, 스토리지 및 네트워킹 리소스를 의미합니다. 인프라는 Amazon EC2 인스턴스를 기반으로 구축되는 서비스 세트입니다. GameLift 없이 자체 EC2 인스턴스를 할당하고, 자체 소프트웨어를 설치하고, EC2 Auto Scaling 그룹을 사용하여 관리할 수 있습니다. 그러나 GameLift는 추가적인 기능 및 이점을 제공합니다. 여기에는 플릿 관리, 게임에 맞게 조정된 자동 크기 조정, 게임에서 스팟 인스턴스를 사용하기 위한 특별 설비가 포함됩니다.
    
    GameLift는 게임 요구 사항을 충족하는 다양한 인스턴스 유형을 제공합니다. 리전별 현재 이용 가능 여부는 온라인 설명서를 참조하십시오.
    
    인프라는 플릿이라고 하는 컨테이너의 모든 인스턴스를 관리합니다. 플릿은 단일 인스턴스 유형으로 정의되며 플릿 내에 포함된 모든 인스턴스는 동일합니다. 플릿은 단일 리전에서 정의됩니다. 플릿은 수평 확장을 지원합니다. 즉, 플릿에 인스턴스를 추가할 수 있습니다. GameLift 대시보드에서 플릿의 인스턴스 수를 변경하여 플릿을 수동으로 확장할 수 있습니다.
    
    플레이어 수요는 변경되기 마련이므로 게임에 사용할 인스턴스 수를 선택하기란 어려운 문제입니다.
    
    인프라 리소스가 너무 적으면 플레이어가 게임을 플레이할 때까지 기다려야 하므로 플레이어 경험이 좋지 않습니다.
    
    리소스가 너무 많으면 사용되지 않는 일부 리소스에 대해 비용을 지불하게 되므로 비용 효율적이지 않습니다.
    
    더 나은 해결책은 플레이어 수요에 따라 인프라를 확장 및 축소하는 것입니다. 게임 세션 용량은 플레이하려는 모든 플레이어를 수용하기에 충분해야 합니다. 스파이크와 같이 증가하는 수요를 처리하기 위해 일부 유휴 용량을 따로 설정해야 합니다. 따라서 온디맨드로 플레이어를 수용하는 데 사용할 수 있는 게임 세션의 버퍼를 준비하는 것이 좋습니다. 총 게임 세션 수는 현재 사용 중인 게임 세션 수보다 약간 높아야 합니다. 플릿의 인스턴스 수를 늘려 전체 용량이 증가하더라도 동일한 비율의 게임 세션 버퍼를 예약하는 것이 좋습니다. 플릿에 인스턴스를 추가하는 조정 이벤트에도 새 인스턴스가 준비되고 게임 서버가 준비되며 게임 세션이 시작될 때까지 시간이 걸립니다. 따라서 게임 세션의 버퍼 비율은 조정 이벤트 중 플레이어 수요를 처리하기에 충분해야 합니다.
    
    대상 추적을 통한 플릿 자동 조정으로 이 솔루션을 구현할 수 있습니다.
    
    대상 추적은 필요에 따라 서버 용량을 추가하거나 제거하여 사용 가능한 게임 세션을 지정한 목표 비율로 유지합니다.
    
    대상 추적은 **“사용 가능한 게임 세션 비율**” **지표를** 사용합니다. GameLift는 사용 가능한 총 게임 세션과 사용 중인 게임 세션 수를 모니터링하고, 플릿에서 인스턴스를 추가 또는 제거하여 사용 가능한 게임 세션의 일정 비율을 유지합니다.
    
    자동 조정은 여러 가용 영역에 인스턴스를 자동으로 할당하여 안정성을 유지합니다.
    
    대상 추적 규칙
    
    사용 가능한 게임 세션 비율에 대한 올바른 값은 플릿 크기에 따라 다릅니다.  작은 크기에서 응답성을 높이려면 더 큰 비율이 필요합니다. 플릿이 크면 절대적인 실제 게임 세션이 더 많이 필요하지만 총 용량의 비율은 작아집니다.
    
    예를 들어
    
    인스턴스 2개와 인스턴스당 게임 서버 2개가 있는 개발 플릿에는 4개의 세션이 있습니다.
    
    하나의 게임 세션을 사용하려면 사용 가능한 게임 세션의 비율이 0%보다 크고 25%보다 작아야 합니다.
    
    2개의 게임 세션을 사용하려면 26%가 필요합니다.
    
    인스턴스가 1,000개이고 인스턴스당 게임 서버가 2개인 프로덕션 플릿에는 2,000개의 세션이 있습니다.
    
    소규모 플릿과 마찬가지로 세션의 25%를 사용하려면 500개 세션을 사용해야 하는데 이는 너무 많습니다.
    
    5% 또는 약 100개 세션이 더 적절합니다.
    
    실제로 게임 개발자는 더 높은 “안전한” 비율/사용 가능한 게임 세션 수에서 시작한 다음 경험에 따라 이 값을 줄입니다.
    
    GameLift는 현재 예약 인스턴스를 지원하지 않습니다. 대기 수요는 대부분의 게임에서 최대 수요의 10%가 될 수 있습니다. 매일 인스턴스를 켜고 끄는 경우 예약 인스턴스를 실행하는 것이 적절하지 않습니다. 1년 또는 3년 계약 기간의 수요는 예측하기 어려울 수 있습니다.
    
    GameLift는 스팟 인스턴스를 지원합니다.
    
    스팟 인스턴스는 유사한 온 디맨드 인스턴스에 비해 낮은 요금으로 제공됩니다. 현재 가격은 시장 입찰 시스템에 의해 결정됩니다. 온디맨드로 구매한 경우 동일한 인스턴스보다 70% 저렴할 수 있습니다. 스팟 플릿을 사용할 경우 GameLift는 플레이어 시간당 비용으로 측정할 때 현재 제공되는 솔루션 중 가장 비용 효율적인 솔루션이 됩니다.
    
    게임 세션에는 스팟 인스턴스가 적합하지 않을 수 있습니다. 게임 세션은 매번 완료될 때까지 실행되어야 합니다. 스팟 인스턴스는 겨우 2분 전 알림으로 회수될 수 있습니다.
    
    이러한 이유로 GameLift에는 스팟 인스턴스를 사용하기 위한 FleetIQ가 도입되었습니다.  FleetIQ 는 스팟 인스턴스 중단 가능성을 측정하는 방법입니다. 스팟 인스턴스 플릿이 중단될 가능성이 낮다고 판단되는 경우에는 게임을 배치할 수 있습니다. FleetIQ 는 스팟 플릿을 평가하고 향후 몇 시간 내에 중단 가능성이 있다고 판단되면 플릿을 유지 불가능한 것으로 표시합니다. 플릿에서 실행 중인 모든 게임은 완료될 때까지 계속 실행됩니다. 그러나 이 플릿에서는 새로운 게임이 시작되지 않습니다. 이 수요는 온디맨드 대체 플릿에 수용됩니다. 나중에 FleetIQ 가 스팟 플릿이 다시 실행 가능하다고 판단하면 가장 저렴한 플릿에서 새로운 게임이 시작됩니다. 게임 세션 배치 대기열은 플레이어와 가장 가까운 리전에서 사용 가능한 플릿 중 가장 저렴한 플릿을 결정합니다.
    
    일반적으로 스팟 플릿을 단독으로 사용하지 않고 항상 동일한 인스턴스 유형의 온디맨드 플릿과 페어링합니다.
    
    선택한 인스턴스 유형은 자동 크기 조정에 영향을 줄 수 있습니다.
    
    플릿 A에는 각각 5개의 게임 서버를 호스팅할 수 있는 소형 인스턴스 유형이 6개 있으며, 총 30대의 게임 서버를 호스팅할 수 있습니다.
    
    플릿 B에는 각각 10개의 게임 서버를 호스팅할 수 있는 대형 인스턴스 유형이 3개 있으므로 30대의 게임 서버를 호스팅할 수 있습니다.
    
    플릿 A와 플릿 B의 한 가지 차이점은 세분화 수준입니다. 인스턴스가 플릿 B에 추가되거나 제거되면 10개의 게임 세션이 추가되거나 제거됩니다.
    
    비용 효율적인 플릿 설계는 의도한 리전에서 인스턴스 유형의 현재 가격에 따라 다를 수 있습니다.
    
    다음은 인스턴스 유형을 선택하는 세 가지 주요 원칙입니다.
    
    원칙 1
    
    일반적으로 인스턴스 유형을 늘리면 규모의 경제가 실현됩니다. 인스턴스 가격이 두 배인 경우 게임 서버 수는 두 배를 넘는 경우가 많습니다. 예를 들어 한 인스턴스 유형이 게임 서버 5개를 제공할 때 비용이 두 배인 다른 인스턴스 유형은 11개의 게임 서버를 제공할 수 있습니다.
    
    원칙 2
    
    스팟 플릿을 사용하려는 경우 일반적으로 높은 중단률을 낮추는 인스턴스 유형을 선택하는 것이 좋습니다.
    
    이 인스턴스 유형은 거의 항상 c4.large입니다. 90% 이상의 고객이 스팟 플릿에 이 인스턴스 유형을 사용합니다.
    
    단일 리전에서 여러 스팟 플릿을 사용할 계획이라면 서로 다른 인스턴스 유형을 사용하는 것이 좋습니다. 한 인스턴스 유형을 사용할 수 없는 경우 다른 인스턴스 유형을 사용할 수 있을 가능성이 높기 때문입니다.
    
    원칙 3
    
    스팟 인스턴스 유형을 사용하려는 경우 온디맨드 인스턴스의 크기를 동일하게 지정할 수 있습니다.
    
    특히, 이후 비용 최적화 단계에서 사용 중인 인스턴스 유형에 적합하게 게임 서버 소프트웨어를 최적화합니다.
    
- 세션 관리 시스템
    
    세션 관리 시스템
    
    GameLift에서 세션은 서로 다르지만 관련이 있는 두 가지 개념을 나타냅니다. 게임 세션은 게임 서버에서 실행되는 게임 시뮬레이션 코드입니다. 게임 세션 관리는 게임을 준비, 시작, 운영 및 종료하는 과정입니다. 또 다른 종류의 세션은 플레이어 세션입니다. 플레이어 세션은 게임을 플레이하기 위한 요청입니다. 플레이어 세션은 플레이어가 게임을 요청할 때 시작되고, 플레이어가 게임 세션에 연결될 때 완료됩니다.
    
    게임 세션 관리
    
    세션 관리에서 게임 세션 배치 대기열이 **CreateGameSession( )**을 호출합니다. 이 호출은 제어를 넘기는 것입니다.
    
    세션 관리는 플릿의 인프라에 게임 세션을 배치합니다. 세션 관리는 가장 최신의 플릿 인스턴스에 게임 세션을 배치하도록 선택합니다. 이렇게 하면 플릿을 축소할 때 **NewGameSessionProtectionPolicy**가 **FullProtection**으로 설정된 경우 현재 실행 중인 게임이 없는 인스턴스에서만 축소가 수행됩니다.
    
    게임 서버 프로세스가 시작되면 게임 세션의 호스팅을 준비하는 설정 코드가 실행됩니다. 그런 다음 유휴 상태로 전환되어 게임 세션의 시작을 알리는 GameLift의 알림을 기다립니다.
    
    게임 세션이 GameLift에 생성된 후에는 어떻게 될까요?
    
    GameLift는 콜백 함수 **onStartGameSession( )**을 구동하고 게임 서버 코드에 **GameSession** 객체를 전달합니다. **GameSession** 객체에는 **GameSessionID**, 플레이어의 최대 수 및 키-값 페어로 저장된 게임 속성 목록이 포함됩니다. 속성의 예로는 게임 모드, 레벨 및 맵이 있습니다. 게임 서버 코드는 요청된 게임 유형을 호스팅할 준비를 합니다. 예를 들어 맵 또는 레벨을 로드합니다. 또한 게임에서 플레이어의 최대 수에 대한 플레이어 슬롯을 준비해야 합니다. 준비가 완료되면 게임 서버는 **ActivateGameSession( )**을 호출하여 게임 세션 상태를 **ACTIVE**로 변경합니다.
    
    GameLift에서 게임은 일정 기간 실행된 다음 종료됩니다. 이 수명을 게임 세션이라고 합니다. 게임 세션의 작동 방식은 게임 서버 소프트웨어를 코딩하는 방법에 따라 다릅니다. 게임 세션을 정상적으로 종료하는 방법은 두 가지입니다. 이러한 방법은 게임 서버 코드에서 GameLift API 호출로 구현됩니다.
    
    첫 번째 방법에서는 게임 서버를 계속 실행하고 게임 세션을 재사용할 수 있습니다. 즉, 게임이 완료되면 코드가 정리를 수행하고 이전 게임 세션을 정리한 후 새 게임을 시작할 준비가 되었음을 GameLift에 알립니다.
    
    두 번째 방법은 게임 세션에서 API를 통해 GameLift에 게임 서버 프로세스를 종료하도록 지시하는 것입니다. 프로세스가 종료됩니다. GameLift는 더 이상 필요한 수의 게임 서버가 없음을 감지합니다. 따라서 빌드의 대체 게임 서버를 새 게임 서버로 만듭니다. 이렇게 해도 새로운 게임 세션이 즉시 생성되지는 않습니다. 그러나 코드의 요청이 수신되면 게임 세션을 시작할 준비가 된 게임 서버가 생성됩니다.
    
    이 두 방법은 게임 솔루션에 서로 다른 특징을 제공합니다. 예를 들어 새 프로세스를 생성할 때 대규모 게임 자산 및 코드를 업로드하는 것보다 이전 게임을 정리하는 것이 더 빠른 경우 게임 세션을 재사용하는 것이 게임을 더 빠르게 제공할 수 있는 방법입니다. 게임 서버를 재활용하면 메모리 누수와 같은 장기적인 문제를 피할 수 있다는 이점이 있습니다. 매 게임에서 새로운 게임 서버를 시작할 때 누수가 심각해져 게임플레이를 방해할 만큼 길게 발생하지 않을 수도 있습니다.
    
    따라서 특정 방법이 다른 방법보다 성능 이점이 있는지 확인하는 실험을 수행해야 할 수 있습니다.
    
    게임 세션 및 게임 서버는 코드 또는 GameLift를 통해 종료될 수 있습니다.
    
    코드로 게임 세션을 종료하는 경우 마지막 작업은 **TerminateGameSession( )**을 호출하는 것입니다   이 함수는 게임 세션이 종료되었음을 GameLift에 알리고 상태를 **TERMINATED**로 변경합니다. 또한 게임 서버가 새로운 게임 세션을 시작할 준비가 되었음을 GameLift에 알립니다.
    
    **ProcessEnding( )**을 호출하면 게임 서버 프로세스와 게임 세션이 종료됩니다. GameLift가 게임 서버 프로세스를 종료할 수도 있습니다. GameLift는 비정상 서버 프로세스가 발견되었거나, 스팟 인스턴스가 중단되었거나, 인스턴스 용량이 축소된 경우 이를 수행합니다. 이러한 작업은 **onProcessTerminate( )** 콜백 함수를 구동합니다.
    
    정상 종료 코드는 **GetTerminationTime( )**을 호출할 수 있습니다. 스팟 인스턴스가 중단된 경우 **GetTerminationTime( )**은 시간만 반환합니다. 그렇지 않은 경우 **onGameSessionTerminate( )**가 호출될 때 최대 30초간 결과를 쓸 수 있습니다.
    
    플레이어 세션 관리
    
    플레이어 세션은 게임을 플레이하려는 플레이어의 요청입니다.
    
    이 요청은 플레이어를 고유하게 식별하는 플레이어 ID와 함께 게임 백엔드로 전송됩니다.
    
    게임 백엔드는 **CreatePlayerSession( )**을 호출하여 플레이어 세션 ID(플레이어 세션 토큰이라고도 함)를 생성합니다. 플레이어 세션 토큰은 플레이어가 게임에 연결할 수 있도록 IP 주소 및 포트 번호와 함께 플레이어의 게임 소프트웨어로 전달됩니다.
    
    핸드셰이크 및 인증은 플레이어의 게임 소프트웨어와 게임 세션 사이에서 발생합니다.
    
    슬롯은 플레이어가 점유할 수 있는 게임의 자리입니다. 슬롯은 핸드셰이크 중 상태를 추적하는 데 사용되는 게임 세션 객체입니다. 플레이어 4명이 필요한 게임에는 4개의 슬롯이 포함됩니다. 최대 플레이어 수가 8명인 게임에는 8개의 플레이어 슬롯이 포함됩니다. 배틀로얄 게임에는 100개의 슬롯이 있을 수 있습니다.
    
    GameLift가 플레이어를 게임 세션에 연결할 준비가 되면 게임 세션에 사용 가능한 플레이어 슬롯이 있는지 묻습니다. 게임 세션의 응답이 긍정적인 경우 GameLift는 예약을 요청합니다. 게임 세션은 플레이어의 게임 액세스를 허용하는 고유 토큰을 반환합니다. 게임 세션은 타이머도 설정합니다. 타이머가 만료될 때까지 플레이어가 연결하여 예약을 가져가지 않으면 플레이어 슬롯 예약이 만료되고 슬롯을 다시 사용할 수 있게 됩니다.
    
    플레이어 세션 토큰은 게임에 연결하는 데 필요한 IP 주소 및 포트 번호와 함께 플레이어의 게임 소프트웨어로 전달됩니다.
    
    플레이어의 게임 소프트웨어는 게임 세션에 직접 연결하여 플레이어 세션 토큰을 전달합니다.
    
    게임 서버는 플레이어 세션 토큰으로 **AcceptPlayerSession( )**을 호출하여 플레이어에게 플레이 권한이 있는지 확인해야 합니다.
    
    게임 세션에서 플레이어 연결이 끊어진 것이 감지되면 코드가 플레이어 ID로 **RemovePlayerSession( )**을 호출하여 GameLift에 알리고 사용 가능한 슬롯 수를 업데이트합니다.
    
- 게임 세션 배치 시스템
    
    게임 세션 배치 시스템
    
    게임 세션 배치는 게임을 호스팅할 리전을 선택하는 과정입니다. 이 과정에서는 플레이어가 리전에 연결하는 데 소요되는 평균 지연, 게임 서버를 통해 제공되는 플릿의 가용성 및 가장 저렴한 비용의 플릿 선택 등 다수의 요인이 고려될 수 있습니다.
    
    플레이어 게임 소프트웨어는 플레이어의 위치와 GameLift 엔드포인트 간의 지연 데이터를 수집해야 합니다.
    
    모범 사례는 플레이어 게임 소프트웨어가 GameLift 엔드포인트의 게임 백엔드에서 목록을 요청하는 것입니다.
    
    플레이어 게임 소프트웨어는 이 작업을 수행하기 위해 ICMP 8(ping)을 각 엔드포인트로 전송합니다. 그러면 지연(밀리초)이 반환됩니다. 이 값은 각 GameLift 플릿에 대한 플레이어 지연을 나타냅니다.
    
    ping을 자주 실행하는 것은 특히 간단한 게임의 경우 비효율적입니다. 따라서 데이터를 캐시하고 슬라이딩 윈도우의 평균을 제공하여 현재 및 일반 조건을 모두 나타내는 지연을 보고하는 것이 좋습니다. 예를 들어 한 가지 해결 방법은 플레이 중에 한 시간에 한 번 ping하는 것이 될 수 있습니다.  그런 다음 최근 5개 데이터 포인트(연속적이지 않을 수 있는 게임플레이 중 최근 5시간)를 사용하여 평균을 계산합니다. 7일간 데이터를 캐시 및 유지하고 7일 후에 이전 데이터를 폐기합니다.
    
    *GameLift 리전 엔드포인트는*
    
    **[https://docs.aws.amazon.com/general/latest/gr/rande.html#gamelift_region](https://docs.aws.amazon.com/general/latest/gr/rande.html#gamelift_region)**
    
    에 게시됩니다.
    
    게임 백엔드는 개별 플레이어 지연 데이터를 수집하고 전달합니다.
    
    매치메이커를 사용하는 경우 지연 데이터는 **StartMatchmaking( )** 요청으로 제출됩니다. 매치메이커는 이 데이터를 사용하고 이 데이터를 게임 세션 배치 대기열에 전달합니다.
    
    매치메이커를 사용하지 않는 경우 **StartGameSessionPlacement( ) 요청으로 게임 세션 배치** 대기열에 직접 요청됩니다.
    
    플릿 선택에서 목표는 매치 평균 지연이 가장 짧은 리전을 선택하는 것입니다.
    
    게임 세션 배치 시스템은 각 플레이어에게 제공된 데이터의 평균을 계산하여 게임 세션의 평균 지연을 계산합니다.
    
    평균이 가장 낮은 영역이 선택됩니다. 일반적으로 이 리전에는 여러 개의 플릿이 포함됩니다. 하나 이상의 스팟 플릿과 이를 백업하는 온디맨드 플릿이 포함됩니다.
    
    해당 리전에서 비용이 가장 낮은 플릿이 선택됩니다. 가용성이 없는 플릿은 무시됩니다. 용량에 도달했거나, 실행 불가능한 스팟 플릿이거나, 장애로 인해 플릿에 연결할 수 없는 경우 플릿을 사용할 수 없습니다.
    
    플릿의 상태는 GameLift가 선택한 시점과 게임 세션을 시작하기 전에 변경될 수 있습니다. 플릿이 선택되면 GameLift가 게임을 세션을 생성하려고 시도합니다. 실패하면 다음 최상의 옵션으로 이동하여 다시 시도합니다.
    
    표시된 예에서 게임 세션 1은 리전 A의 근처의 플릿 A에 연결되고 게임 세션 2는 인근 리전 B의 플릿에 연결됩니다. 리전 B에는 플릿이 여러 개 있으므로 가장 비용이 저렴한 플릿이 선택됩니다. 이 예에서 B2는 스팟 플릿입니다. 리전에는 B2와 B3이라는 2개의 스팟 플릿이 있습니다. 가장 비용이 저렴한 플릿인 B2가 선택됩니다.
    
    플릿은 때때로 기약없이 가용성이 변경됩니다.
    
    개발자는 플레이어 지연 정책을 사용하여 이러한 경우 플릿에 확장할 시간을 주는 대기 기간을 적용할 수 있습니다. 대기 기간을 적용하면 플레이어 지연이 일정 기간을 초과하는 경우 가장 가까운 가용 리전에 게임 세션을 배치하지 않고 더 가까운 플릿을 사용할 수 있을 때까지 대기합니다. 플레이어 지연 정책은 지연에 가장 민감한 게임 외에는 일반적으로 사용되지 않습니다. 그러나 지연에 민감한 게임의 경우 지연 및 게임플레이 경험을 개선하기 위해 플레이어를 최대 1분 정도 대기하게 하는 것을 선호할 수 있습니다.
    
    각 정책은 최대 개별 플레이어 지연(밀리초)과 정책 기간(초)을 선언합니다. 수집은 지연이 가장 짧은 순서로 처리됩니다. 각 정책은 지정된 기간 동안 유효합니다.
    
    플레이어 지연 데이터 없이 게임 세션 배치를 사용할 수 있습니다. 이전에 선언된 게임 세션 배치 프로세스의 대부분을 건너뛰려는 경우 지연 데이터 없이 게임 세션 배치를 사용할 수 있습니다. 예를 들어 지연이 고려되지 않는 게임에서 리전이 미리 결정되어 있고 정렬 또는 가격 비교를 원하지 않는 경우 이 경우 대기열의 다른 플릿 중에서 가용성을 기준으로 대체 플릿을 선택할 수 있습니다. 게임 세션 배치는 대상 목록을 통해 순서대로 진행됩니다.
    
    이 예에서는 2개의 대기열이 있습니다. 게임 백엔드는 게임 세션 1 요청을 대기열 1로, 게임 세션 2를 대기열 2로 보냅니다. 게임 세션 1은 가용성에 따라 플릿 A1, A2 및 마지막으로 플릿 B에 순서대로 배치됩니다. 게임 세션 2는 가용성에 따라 순서대로 B, A1 및 마지막으로 A2에 순서대로 배치됩니다.
    
    이 예에서 A1은 A2보다 저렴하므로 두 경우에서 모두 기본 및 대체로 적용됩니다.
    
    게임 서비스 설계에서 여러 대기열을 사용하는 주된 이유는 리전을 미리 선택하기 위해서입니다.
    
    여러 대기열을 사용하면 글로벌로 확장할 수 있습니다. 예를 들어 대기열 1은 유럽 게임 호스팅용으로 EU(아일랜드) 리전에 있을 수 있으며, 대기열 2는 미국 게임용으로 미국 동부(버지니아 북부)에 위치할 수 있으며, 대기열 3은 일본 게임용으로 아시아 태평양(도쿄) 리전에 위치할 수 있습니다.
    
    매치메이커 및 대기열은 단일 가용 영역에 존재합니다. 이러한 이유로 일반적인 고가용성 설계에서는 동일한 플릿 목록을 포함하는 대기열 페어 1개와 동일한 매치메이킹 규칙을 포함하는 매치메이커 페어 1개를 생성합니다. 매치메이커가 응답하지 않으면 대체 매치메이커로 장애 조치할 수 있습니다. 이때 사용되는 한 가지 방법은 게임 백엔드에서 두 매치메이커에 대한 정보를 바탕으로 문제를 감지하고 둘 사이를 전환하는 것입니다. 또 다른 방법은 Amazon Route 53를 사용하여 현재 매치메이커로 트래픽을 보내는 것입니다. 이렇게 하면 Route 53의 레코드만 업데이트하면 됩니다.
    
    참고로 이 설계는 로드 밸런싱 설계가 아니라 장애 조치 설계입니다. 두 매치메이커-대기열 페어 간의 로드 밸런싱이 필요하지 않은 이유는 매치메이킹 관점에서 사용자가 분할될 경우 최적의 매치를 찾기가 더 어려워지기 때문입니다.
    
- 매치메이킹 시스템
    
    매치메이킹 시스템
    
    GameLift 매치메이킹 소프트웨어를 FlexMatch라고 합니다. 개발자는 다수의 매치메이커를 생성할 수 있습니다. 예를 들어 표준 게임 매치를 생성하는 매치메이커와 고급 매치를 생성하는 매치메이커를 둘 수 있습니다. 매치메이커는 각각 MatchmakingRuleset를 참조하는 MatchmakingConfiguration에 정의됩니다. 규칙 세트는 규칙의 시간대별 수정인 규칙 세트 확장에 의해 수정될 수 있습니다. 마지막으로, 매치메이커는 매치메이킹 요청에 제공된 데이터를 사용합니다. 요청은 일반적으로 여러 플레이어를 대신하여 게임 백엔드를 통해 제출되며, 매치메이킹 규칙에 사용되는 플레이어 속성 데이터와 플레이어 지연 데이터를 포함할 수 있습니다.
    
    매치메이커는 **CreateMatchmakingConfiguration( )**을 사용하여 특정 리전에 생성됩니다. 플레이어 매칭 기준을 정의할때 매치메이킹 규칙 세트를 참조됩니다.
    
    매치는 각 매치메이킹 요청에서 하나로 그룹화된 매치메이킹 티켓 세트이며 플레이어 매칭에 대한 규칙 세트 기준을 충족합니다.
    
    작동 방식은 다음과 같습니다. **StartMatchmakingRequest( )**를 제출합니다. FlexMatch는 해당 요청에 대한 매치메이킹 티켓을 생성하여 플레이어 풀에 추가합니다. 그러면 **MatchmakingSearching** 이벤트가 생성됩니다.
    
    매치메이킹 요청에는 2명 이상의 플레이어가 포함될 수 있으며 이 경우 요청의 모든 플레이어를 함께 고려해야 합니다. 즉, 플레이어가 항상 동일한 매치에 포함되어야 하고 요청의 모든 플레이어가 규칙을 충족해야 티캣이 매칭됩니다.
    
    플레이어 풀의 한 티켓은 앵커로 선택됩니다. 일반적으로 가장 오래된 티켓이 앵커로 선택됩니다. 앵커를 사용하여 매치를 생성하기 위해 나머지 티켓에 대한 철저한 검색이 수행됩니다. 가장 좋은 매치가 형성되면 풀의 모든 매칭 티켓이 제거됩니다. 그러면 **PotentialMatchCreated** 이벤트가 생성됩니다.
    
    매치메이커 구성에 **AcceptanceRequired** 파라미터가 설정된 경우 게임 백엔드에서 **PotentialMatchCreated** 이벤트를 수신합니다. 게임 백엔드는 **AcceptMatch(**)를 호출해야 합니다. 플레이어 중 누군가가 매치를 거부하거나 지정된 제한 시간 전에 수락이 수신되지 않으면 제안된 매치가 삭제됩니다. 매치메이킹 티켓은 다음 두 가지 방법 중 하나로 처리됩니다. 1명 이상의 플레이어가 경기를 거부한 티켓의 경우 티켓 상태가 **SEARCHING** 상태로 돌아가고 새 매치를 찾습니다. 1명 이상의 플레이어가 응답하지 않는 티켓의 경우 티켓 상태가 **CANCELLED**으로 설정되고 처리가 종료됩니다. 필요한 경우 이러한 플레이어에 대한 새 매치메이킹 요청을 제출할 수 있습니다. 모든 플레이어가 경기를 수락하면 **AcceptMatchCompleted** 이벤트가 생성됩니다.
    
    앵커로 매치를 생성할 수 없는 경우 제안된 매치가 삭제되고 다른 앵커가 선택됩니다. 매치메이커는 가능한 한 많은 매치를 만들기 위해 풀을 반복적으로 통과합니다. 티켓이 풀에 들어오고 나가는 속도가 비슷하면 동적 균형이 발생할 수 있습니다. 풀에 남아 있는 티켓이 오래되면 규칙 확장을 사용하여 매칭 규칙을 해제할 수 있습니다.
    
    규칙 확장은 지정된 기간(초) 후에 규칙을 수정하는 단계입니다. 규칙 확장은 제안된 매치의 모든 티켓이 지정된 기간보다 오래된 경우에만 발생할 수 있습니다. 이는 이전 티켓을 사용하여 새로 도착하는 플레이어에게 적절한 매치를 제공하기 위한 것입니다.
    
    매치가 생성되면 **MatchmakingSucceeded** 이벤트가 발생합니다. 매치메이커에 지정된 기간 내에 매치메이킹 티켓이 매칭되지 않으면 **MatchmakingTimedOut** 이벤트가 발생합니다.
    
    이 예에서는 매치메이킹 정책에 따라 정확히 4명의 플레이어가 있어야 합니다.
    
    1번의 경우 FlexMatch는 플레이어 2명이 포함된 요청을 수신하고 이 2명의 플레이어에 대한 매치 티켓을 생성합니다. 다른 2명의 플레이어에 대한 두 번째 요청을 수신하고 이 2명의 플레이어에 대한 매치 티켓을 생성합니다. 티켓은 풀 안에 있습니다. 규칙은 반복적으로 적용됩니다. 가장 오래된 티켓은 앵커로 사용되며 다른 티켓은 앵커와 비교됩니다. 기준을 충족하면 매치가 형성됩니다.
    
    2번의 경우 FlexMatch는 3명의 플레이어가 포함된 요청과 1명의 플레이어가 포함된 다른 요청을 수신합니다. 가장 오래된 티켓은 앵커로 사용되며 다른 티켓은 앵커와 비교됩니다. 기준을 충족하므로 일치가 형성됩니다.
    
    3번의 경우 FlexMatch는 플레이어 3명이 포함된 요청과 플레이어 2명이 포함된 다른 요청을 수신합니다. 이 경우 숫자가 4인 규칙을 충족하지 않으므로 매치를 형성할 수 없습니다.
    
    매치메이킹 규칙 세트는 매치메이커가 자체 구성 이상으로 작동하는 방식을 정의하는 JSON 문서입니다. 규칙 세트에는 플레이어 속성, 게임에서 허용되는 팀 구성, 플레이어 매칭 규칙, 일정 기간 후에 규칙을 해제하기 위한 확장이 포함됩니다. 규칙 세트에는 두 가지 유형이 있습니다. 하나는 플레이어 수가 최대 40명인 “소규모 매치”라고 하는 유형이고, 다른 하나는 배틀로얄 스타일의 게임을 위한 “대규모 매치”라는 유형으로 이 유형의 플레이어는 최대 200명입니다. 대규모 매치에는 플레이어를 배치하기 위한 알고리즘이 추가적으로 포함되어 있으며 규칙 수를 단 1개로 제한합니다.
    
    대규모 매치의 알고리즘을 사용할 때는 유사한 플레이어로 더 큰 매치를 만드는 데 높은 우선 순위를 두거나 모든 플레이어에게 가능한 최상의 플레이어 지연 경험을 제공하는 매치를 생성하는 것 중에서 선택할 수 있습니다.
    
    개발자는 플레이어 속성을 정의하는 문서를 작성하여 각 플레이어에 대해 전달되는 정보를 설명하며, 매치 결정은 이 정보를 바탕으로 내려집니다.
    
    각 플레이어 속성은 "string", "number", "string_list" 또는 "string_number_map" 중 하나이며 플레이어가 플레이하고자 하는 맵, 선택한 게임플레이 모드, 선호하는 팀 또는 역할 또는 플레이어가 선택한 캐릭터 클래스 등을 나타냅니다.
    
    티켓의 플레이어가 매치의 요구 사항을 충족하는지 여부를 확인하기 위해 플레이어 속성 데이터를 사용하여 티켓에 규칙이 적용됩니다.
    
    속성 표현식은 속성 데이터를 규칙으로 평가될 수 있는 형식으로 처리하는 표현식입니다.
    
    게임 백엔드는 결합된 요청을 매치메이킹 시스템에 제출하기 전에 플레이어 요청을 수집하고 처리합니다. GameLift Client SDK를 플레이어의 게임 소프트웨어에 포함하지 않고 게임 백엔드를 생성하는 데는
    
    여러 가지 이유가 있습니다.
    
    첫 번째는 보안입니다. GameLift API는 함수별 인증을 구현하지 않습니다. 백엔드의 인증 프로세스를 사용하여 게임 백엔드와 매치메이킹 간의 통신을 보호할 수 있습니다.
    
    또 다른 이유는 매치메이킹 요청의 일괄 처리와 관련이 있습니다. 예를 들어 일반적으로 4명의 플레이어가 있지만 4명이 없을 때 2명의 플레이어를 허용할 수 있는 게임에서 각 플레이어가 매치메이킹에 직접 요청을 제출할 수 있다면 플레이어가 2명인 게임이 많이 발생할 수 있습니다. 매치메이커는 단순히 개별 플레이어가 요청할 때보다 빠르게 매치를 생성할 수 있습니다. 게임 백엔드가 요청을 수집하고 4명의 플레이어가 있는 단일 티켓을 제출하면 매치메이커는 4인 플레이어 게임을 만듭니다.
    
    또 다른 이유는 속성 값을 사전 처리하거나 해석하기 위해서입니다. 예를 들어 절대 속성 값을 백분율로 변환하거나 로그 척도로 해석할 수 있고, 또는 속성 값을 히스토그램에 매핑할 수 있습니다. 이렇게 하면 수요의 급격한 변화와 특이점을 가진 속성을 처리할 수 있습니다.
    
    팀
    
    팀 크기가 같을 필요는 없습니다. 팀은 여러 유형의 캐릭터나 역할로 구성될 수 있습니다. 예를 들어, 각 팀에는 마법사 전사, 성직자, 도둑이 필요할 수 있습니다.
    
    규칙에는 6가지 유형이 있습니다.
    
    거리 규칙은 플레이어 속성 값을 지정된 범위 내에 유지합니다.
    
    비교 규칙은 플레이어 속성 값 간의 논리적 비교를 허용합니다.
    
    컬렉션 규칙은 플레이어 속성 값 세트 간의 교차점을 결정하는 데 사용됩니다.
    
    지연 규칙은 가장 가까운 리전이 동일한 플레이어의 매칭을 허용하여 모두에게 짧은 지연을 제공합니다.
    
    거리 정렬 규칙은 앵커 요청으로부터의 거리를 기준으로 앵커 요청의 거리에 따라 플레이어 속성을 기준으로 매치메이킹 요청을 미리 정렬합니다.
    
    절대 정렬 규칙은 플레이어 속성 값을 기준으로 매치메이킹 요청을 오름차순 또는 내림차순으로 미리 정렬합니다.
    
    이 규칙은 계산된 팀 속성을 기반으로 합니다.
    
    **count(teams[cowboys].players)**는 "카우보이 팀과 플레이어 목록을 가져오고 플레이어 수를 계산"할 것을 지시하는 속성 표현식입니다. 이 식은 하나의 피연산자가 됩니다.
    
    또다른 속성 표현식인 **count(teams[aliens].players)**는 다른 피연산자를 제공합니다.
    
    이 규칙은 개별 플레이어 속성을 기반으로 합니다. 이 예에서 속성은 플레이어 스킬입니다.
    
    **avg(teams[*].players.attributes[skill])**은 "각 팀, 각 팀의 모든 플레이어 및 스킬 속성을 가져온 다음 이 모든 속성의 평균을 계산"할 것을 지시하는 속성 표현식입니다. 이렇게 하면 각 팀의 평균 스킬 레벨이 생성됩니다. 이것이 하나의 피연산자가 됩니다.
    
    또 다른 정리 속성 표현식인**avg(flatten(teams[*].players.attributes[skill]))**은 다른 피연산자를 제공합니다.
    
    이 규칙은 각 팀 스킬 값이 모든 팀의 결합된 전체 스킬 값에서 10의 거리 내에 있는지 테스트합니다. 이로써 공정한 매치가 생성됩니다.
    
    매치메이킹 제어
    
    매치메이킹 지연 규칙은 허용 가능한 매치에 대한 지연을 특별히 제한합니다.
    
    예를 들어, 매치된 모든 플레이어들의 리전 지연이 반드시 최대 한도 이내여야 한다고 규칙으로 정할 수 있습니다. 현재 이 규칙 유형은 대규모 매치에 대한 규칙 세트에 사용할 수 있는 유일한 규칙 유형이며 maxLatency 설정만 사용할 수 있습니다.
    
    한 리전 주변에 있는 플레이어들이 함께 매칭될 가능성이 증가하므로 모두가 짧은 지연으로 게임을 플레이할 수 있습니다.
    
    규칙 확장은 지정된 기간(초) 후에 규칙을 수정하는 단계로 정의됩니다. 규칙 확장은 제안된 매치의 모든 티켓이 지정된 기간보다 오래된 경우에만 발생할 수 있습니다. 이는 이전 티켓을 사용하여 새로 도착하는 플레이어에게 적절한 매치를 제공하기 위한 것입니다.
    
    위의 예에서 고려된 FairTeamSkill 규칙은 일정 기간 후에 확장됩니다.
    
    티켓이 생성되면 원래 규칙에 따라 팀 기술이 평균 팀 스킬에서 최대 10까지의 거리에 있는 다른 티켓과 매칭될 수 있습니다.
    
    티켓이 5초가 되면 팀 스킬이 평균 팀 스킬에서 최대 50까지의 거리에 있는 다른 티켓과 매칭될 수 있습니다.
    
    티켓이 20초가 되면 팀 스킬이 평균 팀 스킬에서 최대 100까지의 거리에 있는 다른 티켓과 매칭될 수 있습니다.
    
    게임에는 최대 플레이어 슬롯 수가 있습니다. 매치메이킹 중에 모두 채워질 수도 있고 아닐 수도 있습니다. 다시 말해, 일부 게임은 플레이어의 전체 보충 없이 시작될 수 있습니다.
    
    두 번째 시나리오에서는 플레이어가 전체 게임에서 나갈 경우 빈 슬롯을 다시 채울 수 있습니다.
    
    채우기는 실행 중인 게임의 플레이어와 매치메이킹 풀의 플레이어를 매칭하여 게임의 추가 플레이어를 찾는 과정입니다. 채우기의 단일 패스로 게임 플레이어 슬롯이 완전히 채워질 수도 있고 채워지지 않을 수도 있습니다.
    
    채우기는 **MatchmakingConfiguration**에 정의된 대로 서버에서 시작되거나 자동으로 시작될 수 있으며 각각의 프로세스는 서로 다릅니다. 자동 채우기는 첫 번째 채우기 시나리오에만 적용되며 플레이어를 완전히 보충하지 않고 시작되는 게임을 채웁니다. 활성화하면 매치가 생성된 후 플레이어가 완전히 보충될 때까지 채우기 패스가 수행됩니다. 게임에 플레이어가 추가될 때마다 서버는 게임 서버 소프트웨어 내에서 신규 및 기존 플레이어를 포함한 전체 플레이어 목록과 함께 **updateGameSession( )** 콜백을 수신합니다.
    
    수동 채우기는 서버에서 시작되는 채우기와 달리 두 채우기 시나리오에 모두 적용됩니다. 서버는 **StartMatchBackfill( )**을 통해 채우기를 요청할 때마다 현재 모든 플레이어의 매치메이커 데이터를 가져오고 해당 데이터를 GameLift로 전달합니다. 그러면 매치메이킹 티켓이 생성되고 풀에 배치됩니다. 매치메이킹 프로세스는 평소와 같이 진행되며, 채우기 티켓을 다른 티켓과 결합하여 새로운 매치를 만들 수 있습니다. 매치가 생성될 때마다 게임 서버는 게임 서버 소프트웨어 내에서 신규 및 기존 플레이어를 포함한 전체 플레이어 목록과 함께 **updateGameSession( )** 콜백을 수신합니다.
    
    각각의 경우에 게임 백엔드는 연결 정보가 포함된 매치메이킹 티켓에 대한 SNS 알림을 수신하며, 이 정보는 게임 백엔드에 의해 플레이어에게 전달되어 게임 세션에 참가하는 데 사용됩니다.
    
    GameLift FlexMatch 이벤트는 SNS 알림을 통해 전달될 수 있습니다. 게임 백엔드 소프트웨어와 같은 클라이언트는 이러한 알림을 구독하고 이를 사용하여 이벤트에 응답할 수 있습니다.
    
    FlexMatch 매치메이킹 이벤트
    
    MatchmakingSearching은 매치메이킹 풀에 티켓을 입력할 때 생성되는 이벤트입니다.
    
    PotentialMatchCreated는 티켓이 규칙 세트를 만족하고 매치가 형성될 때 생성되는 이벤트입니다. 매치메이커가 플레이어 게임 수락을 요구하는 경우 백엔드는 이 시점에서 이 이벤트를 수신해야 합니다. 그러나 이 이벤트는 매치메이커가 게임 수락을 요구하는지 여부에 관계없이 생성됩니다.
    
    AcceptMatch는 플레이어가 잠재적 매치를 수락할 때 발생하는 이벤트입니다. 매치에 있는 플레이어의 현재 수락 상태가 포함됩니다.
    
    AcceptMatchCompleted는 플레이어 수락이 완료되거나, 플레이어가 거부하거나, 수락 시간이 초과될 때 발생합니다. 매치의 최종 수락 상태가 포함됩니다.
    
    MatchmakingSucceeded는 매치메이킹이 성공적으로 완료되고 게임 세션이 생성되었을 때 발생합니다.
    
    MatchmakingTimedOut은 MatchmakingConfiguration의 **RequestTimeoutSeconds** 속성보다 오래된 티켓이 만료될 때마다 발생합니다.
    
    MatchmakingCancelled는 일반적으로 게임 백엔드에서 **StopMatchmaking( )**을 호출하여 매치메이킹 티켓이 취소된 경우에 발생합니다.
    
    MatchmakingFailed는 FlexMatch 내부 문제로 인해 티켓이 폐기되는 경우 발생합니다. **GameSessionPlacementQueue**에 연결할 수 없는 경우가 한 예입니다.
    
    이벤트를 사용하려면 개발자가 SNS 주제를 생성한 다음 Lambda와 같은 호환되는 리소스를 사용하여 구독해야 합니다.
    
    FlexMatch 이벤트는 매치메이킹 요청에 대한 정보를 수신할 때 우선적으로 사용되는 방법입니다. 낮은 TPS(초당 트랜잭션 수)를 위해 설계된 **DescribeMatchmaking( )**과 같은 함수보다 확장성이 훨씬 뛰어나기 때문입니다.
    
    실제로 채우기를 수행할 때는 거의 SNS 주제를 사용하여 게임 백엔드에서 매치메이커를 지속적으로 폴링하는 일이 없도록 해야 합니다.
    
- GameLift 개발 개요
    
    GameLift 개발 개요
    
    이 단원에서는 GameLift를 사용하여 게임 서비스를 개발하는 방법을 배웁니다. GameLift의 일부를 게임 서버 소프트웨어와 통합하고 GameLift의 다른 일부를 게임 백엔드 클라이언트 소프트웨어와 통합해야 합니다. 먼저 플레이어의 게임 소프트웨어가 게임 백엔드와 통신하는 방법을 살펴보면서 전체적인 그림으로 전개하도록 하겠습니다.
    
    플레이어 구성 요소
    
    플레이어는 플레이어의 소프트웨어를 구성하는 게임 사용자 인터페이스를 통해 게임을 플레이합니다. 게임 개발자는 보통 Unity(C# 사용) 또는 Unreal Engine이나 Lumberyard(C++ 사용)와 같은 게임 엔진을 사용하여 이 구성 요소를 작성합니다. 플레이어 소프트웨어는 컴퓨터, 게임 콘솔, 태블릿, 전화 또는 가상 현실 기어와 같은 플레이어 게임 디바이스에서 실행됩니다.
    
    플레이어의 디바이스와 플레이어 소프트웨어는 일반적으로 GameLift와 직접 통신하지 않습니다. 대신 게임 디바이스는 게임 백엔드라고 하는 별도의 소프트웨어와 통신합니다. 일부 설명서에서는 클라이언트 또는 클라이언트 서비스라고도 하지만 이 과정에서는 게임 백엔드라고 합니다. 게임 백엔드는 일반적으로 웹 서버입니다. 게임 백엔드는 다수의 웹 지향 언어로 작성될 수 있습니다. 일반적인 언어로는 C#, Javascript, Java, Node.js, PHP, Python 및 Ruby가 있습니다. 덜 일반적이지만 가능한 언어로는 C++과 Go가 있습니다. 게임 백엔드는 플레이어 인증을 통해 플레이어에게 게임 리소스를 사용할 수 있는 권한이 있는지 확인합니다. 게임 백엔드는 멀티플레이어 게임을 플레이하려는 개별 플레이어의 요청을 수신합니다. 또한 GameLift API를 통해 GameLift와 통신하여 플레이어를 위한 게임을 주선합니다. 게임 백엔드에는 게임 엔진의 요소가 포함되지 않습니다.  게임 백엔드의 역할은 플레이어를 게임에 연결하는 것입니다.
    
    게임 백엔드는 게임을 원하는 플레이어와 플레이어를 위해 준비된 게임을 연결하는 브로커처럼 작동합니다.
    
    플레이어가 게임 세션에 연결된 후에는 관여하지 않습니다. 이후에 플레이어의 게임 소프트웨어와 게임 디바이스는 실행 중인 게임 세션과 직접 통신합니다. 게임 세션은 게임 서버 소프트웨어의 실행 중인 인스턴스입니다. 게임 세션이 어디서 시작되는지 봅시다.
    
    서버 소프트웨어 통합
    
    게임 서버 소프트웨어는 개발자가 작성합니다. 이 게임 서버 소프트웨어는 일반적으로 C# 또는 C++로 작성됩니다. Unity 개발자는 C#을 사용하며 Unreal Engine 및 Lumberyard 개발자는 C++를 사용하는 경향이 있습니다. GameLift는 C++ 또는 C#을 사용하는 다른 게임 엔진과도 호환됩니다. 게임 서버 소프트웨어에는 일반적으로 게임 엔진의 요소가 포함되지만 반드시 필요한 것은 아닙니다. 게임 서버 소프트웨어는 일반적으로 백엔드 게임 개발자가 작성합니다. 일반적으로 Windows Server 운영 체제에서 실행하는 것을 목표로 하지만, 백엔드 게임 개발자들의 경우 저렴한 라이선스 비용 덕에 대규모로 실행할 경우 상당한 비용을 절감할 수 있다는 이점 때문에 Linux를 채택하는 추세가 있습니다. 게임 서버 소프트웨어에는 게임이 실행 중일 때 GameLift와 통신할 수 있는 Amazon GameLift SDK가 포함되어야 합니다.
    
    게임 서버 소프트웨어는 파일의 디렉터리입니다. 이 디렉터리에는 실행 파일, 초기화 스크립트 및 실행 파일에 필요한 모든 종속 소프트웨어 또는 라이브러리가 포함되어야 합니다. 게임 서버 소프트웨어를 GameLift에 업로드한 후 게임 서버 소프트웨어는 GameLift 빌드라는 단일 객체가 됩니다. 빌드 객체는 불투명합니다. GameLift 대시보드에서 빌드의 콘텐츠를 볼 수 없습니다. GameLift는 유효성 검사 프로세스를 수행한 후 빌드가 최소 기준(예: 실행 파일 포함)을 충족하는 것으로 확인되면 [Ready]로 표시합니다.
    
    빌드는 인스턴스 플릿을 정의할 때 사용됩니다.
    
    GameLift에 플릿을 생성하도록 요청하면 첫 번째 단계로 EC2 시드 인스턴스가 생성됩니다. 이 인스턴스는 빌드의 복사본과 함께 로드됩니다.
    
    프로토타입 인스턴스를 사용하여 인스턴스의 스냅샷인 Amazon Machine Image(AMI)가 생성됩니다. 새 인스턴스는 기본 운영 체제에서 설치 스크립트를 사용하여 시작되는 것이 아니라 AMI에서 생성됩니다.
    
    AMI를 사용하면 많은 소프트웨어가 이미 설치되어 있기 때문에 인스턴스가 더 빨리 시작된다는 이점이 있습니다. 다른 이점으로는 사전 설치된 소프트웨어를 다운로드 할 필요가 없고 소프트웨어 버전이 이미 설정되어 있기 때문에 안정적이라는 것이 있습니다.
    
    AMI가 생성되면 GameLift가 플릿을 구성합니다. 플릿 생성의 마지막 작업은 플릿 내에서 첫 번째 EC2 인스턴스를 시작하는 첫 번째 조정 이벤트를 트리거하는 것입니다. AMI에서 인스턴스가 구축되므로 이미 로컬 파일 시스템에 빌드가 있습니다.
    
    플릿에서 TLS가 활성화된 경우 AMI를 생성하는 동안 플릿에 대한 TLS 인증서가 생성되어 AMI에 포함됩니다.
    
    인스턴스를 초기화하는 동안 하나 이상의 게임 서버 프로세스가 시작됩니다. 게임 서버 프로세스 또는 게임 서버는 빌드에 포함된 게임 서버 소프트웨어를 실행합니다. 게임 서버는 GameLift API 호출을 통해 GameLift와 통합하기 위해 여러 기능을 수행해야 합니다. 이 정리 작업을 완료한 후에는 유휴 상태로 전환됩니다. 이 시점에서 소프트웨어가 로드되고 프로세스에서 실행되지만 아직 게임 세션은 존재하지 않습니다.
    
    GameLift의 일부(인스턴스의 GameLift 에이전트)는 게임 서버 프로세스의 수를 모니터링합니다. 그 중 하나는 하트비트 상태 확인입니다. 프로세스 상태 확인이 세 번 누락되면 프로세스가 중단된 것으로 간주되어 종료 및 교체됩니다. 플릿을 정의할 때는 인스턴스에 있어야 하는 게임 서버의 수를 지정합니다. 실행 중인 게임 서버가 충분하지 않은 경우 GameLift는 사용자가 지정한 수를 유지하기 위해 더 많은 게임 서버를 시작합니다.
    
    인스턴스에서 고정된 수의 게임 서버를 유지하는 것은 인프라 관리 시스템의 가장 중요한 부분입니다.
    
    GameLift API 요청이 있으면 GameLift가 게임 서버에 게임 세션을 시작할 것을 요청합니다. 세션은 게임을 시작하기 전에 초기화 및 설정을 수행합니다. 마지막 시작 작업은 API를 통해 플레이어를 받을 준비가 되었음을 GameLift에 알리는 것입니다.
    
    왼쪽에 서비스 소프트웨어를 게임 서버와 통합하는 데 필요한 단계가 나와 있습니다. GameLift Server SDK를 다운로드합니다. 구축합니다. 라이브러리를 서버 소프트웨어에 연결합니다. 그런 다음 표시된 것과 같은 필수 함수 호출을 구현해야 합니다.
    
    필수 함수는 **InitSdk( )**, **ProcessReady( )**, **ActivateGameSession( )** 및 **ProcessEnding( )**을 포함하여 4개입니다. 필수 콜백은 **onStartGameSession( )**, **onProcessTerminate( )** 및 **onHealthCheck( )의 3개입니다.** 이 함수 그룹은 전체 게임 서버 및 게임 세션 수명 주기를 정의합니다. 설명서에서 이러한 함수에 대해 자세히 알아볼 수 있습니다.
    
    클라이언트 소프트웨어 통합
    
    GameLift를 사용할 때의 핵심 중 하나는 GameLift SDK가 다양한 추상화 수준에서 API 기능을 지원한다는 것입니다. 게임에 매치메이킹 및 게임 세션 배치가 필요한지 여부와 GameLift를 사용하여 이러한 서비스를 제공할지 아니면 코드에서 다른 방식으로 제공할지를 결정해야 합니다. 매치메이킹을 선택하면 매치메이킹 서비스가 하위 수준 서비스를 포함하고 사용하기 때문에 매치메이킹 아래의 수준과 직접 통신할 필요가 없습니다. 게임 세션 배치와 세션 관리도 마찬가지입니다. 게임 세션 배치로 작업하는 경우 세션 관리와 직접 통신할 필요가 없습니다. 사용할 추상화 수준에 대한 결정에 따라 API 사용을 정렬, 집중 및 간소할 수 있습니다.
    
    매치메이킹은 **StartMatchmaking( )**, **DescribeMatchmaking( )** 및 **StopMatchmaking( )**이라는 3개의 함수를 사용합니다.
    
    요청자에게 매치 상태를 알릴 때는 어떻게 할까요? 호출은 비차단(논-블로킹)입니다.**DescribeMatchmaking( )**을 호출해야 할 수 있습니다. 보다 일반적인 접근 방식은 Amazon SNS 단순 알림 서비스를 사용하는 것입니다. 게임 백엔드에서 SNS 주제를 구독합니다. FlexMatch 매치메이커가 주제에 이벤트를 게시합니다. 관심 있는 이벤트가 표시되면 **DescribeMatchmaking( )**을 호출하여 현재 상태를 가져옵니다.
    
    게임 세션 배치를 사용하려면 하나 이상의 게임 세션 배치 대기열을 만들어야 합니다.
    
    GameLift의 매치메이킹 시스템을 사용하는 경우, 게임 세션 배치 함수가 시스템 내부에서 구동되므로 직접 호출할 필요가 없습니다. 자체 매치메이킹 소프트웨어를 구현한 경우 해당 소프트웨어에서 **StartGameSessionPlacement( )** 및 **DescribeGameSessionPlacement( )**를 호출할 수 있습니다. 마지막으로, 코드로 SNS 알림을 구독하여 GameLift 이벤트를 인식할 수 있습니다.
    
    마찬가지로 GameLift의 매치메이킹 시스템 또는 게임 세션 배치 시스템을 사용하는 경우 세션 관리 함수가 내부에서 구동됩니다.
    
    배치 소프트웨어를 직접 작성한 경우 **SearchGameSessions( )**를 호출하여 기존 게임을 찾고 **CreateGameSession( )**을 호출하여 새 게임을 시작할 수 있습니다. **CreatePlayerSessions( )**를 호출하여 플레이어 요청에 대한 슬롯을 예약할 수 있습니다.
    
    Realtime Server Client API 통합
    
    Realtime Server Client API는 플레이어 게임 소프트웨어에 내장됩니다.
    
    이 API는 Realtime 게임 서버에 기본적으로 포함된 구성 요소와 통신합니다.
    
    Realtime 클라이언트는 Realtime 게임 세션에 대한 2개의 통신 채널을 엽니다.
    
    하나는 속도를 위한 UDP입니다. 다른 하나는 안정성을 위한 TCP입니다. 대부분의 통신에서 클라이언트는 UDP를 먼저 시도하고 작동하지 않으면 TCP를 대신 사용합니다.
    
    클라이언트는 opCode를 포함하는 메시지를 보냅니다. opCode는 서버 측에서 수행할 작업을 정의합니다.
    
    서버에서 opCode 맵은 코드를 스크립트에 연결하고 실행을 트리거합니다.
    
    따라서 통신의 양측, 즉 opCode를 보내는 클라이언트와 그 결과 실행되는 서버 측 스크립트를 제어할 수 있습니다.
    
    동기식 메시지는 게임 세션 연결 및 연결 해제와 게임플레이 중에 클라이언트 간에 메시지를 전달하는 데 사용됩니다.
    
    비동기식 메시지는 클라이언트가 서버 측 이벤트에 응답하는 콜백에 사용됩니다.
    
- 게임 데이터
    
    비즈니스 운영 및 의사 결정에 데이터가 미치는 영향은 매우 큽니다. GameLift는 라이브 게임 운영, 게임 설계 및 비즈니스 의사 결정에 필요한 데이터를 제공합니다.
    
    GameLift는 GameLift 대시보드에 데이터를 표시합니다. 여기에는 모든 게임 빌드의 카탈로그가 포함됩니다. 플릿에 대한 세부 정보도 표시됩니다. 게임 세션과 플레이어 세션에 대한 정보가 포함되며
    
    강력하고 구성 가능한 지표 그래프 시스템이 포함되어 있습니다. 이 모든 정보는 게임 서비스를 살펴보고 상태를 파악하는 데 도움이 됩니다. 또한 게임 서비스 운영 절차를 생성하고 라이브 운영을 위한 플레이북을 정의하는 데도 유용합니다.
    
    GameLift는 특정 지표를 Amazon CloudWatch로 전달합니다. 여기에는 플릿, 대기열 지표 및 매치메이킹 관련 지표가 포함됩니다. CloudWatch에서 광범위한 기능 보기를 통해 운영 데이터를 모니터링 및 수집하고 이를 다른 AWS 서비스 또는 온프레미스 서버에서 제공하는 정보와 통합할 수 있습니다.  CloudWatch는 GameLift에 기본적으로 포함된 그래프 기능을 뛰어넘습니다. 예를 들어 GameLift에서는 라이브 운영을 모니터링할 수 있습니다. CloudWatch에서는 지표를 사용하여 비율, 지표 및 통계를 계산하고 이러한 값을 기준으로 그래프를 표시할 수 있습니다.
    
    GameLift는 API 작업을 기록하는 AWS CloudTrail과도 통합됩니다.
    
    CloudTrail은 거버넌스, 규정 준수 및 감사를 위해 설계되었습니다.
    
    CloudTrail이 캡처하는 GameLift 데이터를 사용하여 보고서를 생성할 수 있습니다.
    
    이 보고서를 보관하고 S3를 통해 공유하거나 보고서를 사용하여 Amazon SNS 단순 알림 서비스로 직원에게 알림을 보낼 수 있습니다.
    
    이제 GameLift 데이터 지원을 살펴보겠습니다.
    
    대시보드는 게임 서비스 모니터링을 위한 데이터를 제공합니다. Amazon CloudWatch와 통합하면 통계 분석을 수행할 수 있습니다.
    
    Amazon CloudTrail과 통합하면 거버넌스 및 감사를 수행할 수 있습니다.
    
    마지막으로, GameLift에서 생성되는 데이터는 API를 통해 액세스할 수 있습니다. 즉, 이러한 데이터를 게임 기술 분석 파이프라인에 통합하고 스트리밍 또는 배치 데이터 분석에 사용하거나 기계 학습과 결합하여 고급 분석 및 의사 결정에 활용하고 인공 지능 기능을 게임 서비스에 추가할 수 있습니다.
    
- 게임 재무
    
    게임 금융은 비용에 영향을 줄 수 있는 변경 사항을 경고하고 대응하기 위해 비용 및 기능을 예측, 추정 및 추적하는 데 사용할 수 있는 기능을 말합니다.
    
    재무 정보는 게임 서비스 비즈니스에서 수익성을 최적화하는 데 매우 중요합니다.
    
    GameLift의 첫 번째 재무 기능 범주는 비용 제어입니다.
    
    비용 제어에는 두 가지 종류의 변수가 있습니다. 제어할 수 있는 종속 변수와 제어할 수 없는 독립 변수입니다.
    
    기본적인 독립 변수는 플레이어 수요이며 전적으로 외부 요인에 따라 크게 변동될 수 있습니다.
    
    다른 독립 변수로는 인터넷의 안정성 문제로 인한 네트워크 가용성과 시장 요율에 따라 변동되는 스팟 인스턴스와 같은 리소스 비용이 포함됩니다.
    
    종속 변수에는 게임 서비스를 비즈니스 환경에 맞게 조정하는 데 사용할 수 있는 GameLift의 모든 옵션과 설정이 포함됩니다. 여기에는 플릿 설계, Auto Scaling, 스팟 인스턴스, Realtime 서버, FleetIQ 및 FlexMatch가 포함됩니다.
    
    재무 기능의 두 번째 범주는 가격 및 비용 정보입니다. 가격은 API를 통해 게시되고 제공됩니다. 즉, 현재 가격을 게임 기술 분석 파이프라인에 통합할 수 있습니다. 대시보드에는 3개월의 과거 가격 책정 데이터가 저장되므로 향후 가격을 예측하고 정보에 입각한 결정을 내리는 데 유용할 수 있습니다. GameLift 인스턴스 요금을 추정하는 데 사용할 수 있는 EC2 비용 추정 계산기가 있습니다. GameLift는 계정 할당량, 예산 생성 기능, 비용 워터마크에 도달했을 때 알려주는 조건부 알림 설정 기능 등 모든 AWS 재무 기능을 상속합니다.
    
    GameLift는 AWS 비용 관리 콘솔과 통합됩니다.
    
    여기에서 정교한 리소스 사용 보고서를 생성하고 S3 버킷을 통해 공유할 수 있습니다.
    
    이러한 보고서는 지출과 별도로 리소스 사용을 식별하여 게임 서비스에서 리소스가 사용되는 방식을 분석하는 데 유용할 수 있습니다.
    
    GameLift 재무 지원을 검토하겠습니다.
    
    GameLift의 구성 가능한 대부분의 기능은 비용 관리, 우수한 게임플레이 또는 둘 다를 가능하게 하기 위한 것입니다.
    
    GameLift는 변화하는 다양한 비즈니스 우선 순위를 지원할 수 있는 유연성을 갖추고 있습니다.
    
    GameLift는 가격 투명성을 제공하므로 종속 변수와 독립적으로 가격을 손쉽게 분리하여 보다 정확한 비용을 추정할 수 있습니다.
    
    마지막으로, GameLift를 AWS 비용 관리 콘솔과 통합하여 정교한 재무 내역을 보고할 수 있습니다.
    

### Step 13.2. Matchmaking 구현 시 권고 사항

Game Client에서 Lambda를 통해 StartMatchmaking API를 호출 한 후 폴링 방식으로 DescribeMatchmaking API 를 호출하는 경우, 실 서비스 환경에서 Throttling이 발생할 수 있습니다. 

![readme/Match-001.png](readme/Match-001.png)

그래서 GameLift 쪽으로 DescribeMatchmaking API를 폴링 방식으로 호출하는 것이 아니라

아래와 같이 다른 곳으로 Match Status 결과를 저장하고 그 곳을 확인 하는 것을 권장하고 있습니다.

![readme/Match-002.png](readme/Match-002.png)

다음과 같이 SNS Topic을 생성하고 MatchEvent API 구독하도록 구성합니다.

![readme/Match-003.png](readme/Match-003.png)

그리고 다음과 같이 매치메이킹 구성 시 Notification 대상으로 SNS ARN을 입력합니다.

![readme/Match-004.png](readme/Match-004.png)

위와 같이 구성 시 GameLift 매치가 SNS Topic 으로 바로 푸시되며, Lambda 함수가 이를 받아서 처리할 수 있습니다. 

### Step 13.3. Fleet 인스턴스 구성에 대한 Best practices

Spot Fleet을 서로 다른 2개 타입으로 구성하면 GameLift 의 FleetIQ 알고리즘에 따라 Spot Interruption 이 가장 덜 발생할 것으로 추정되는 Spot Fleet 으로 게임 세션이 배치 됩니다. 

여기에 On-demand Fleet을 하나 두시면 게임 자체의 안정성을 크게 올리실 수 있습니다.

[https://docs.aws.amazon.com/ko_kr/gamelift/latest/developerguide/queues-best-practices.html#queues-design-spot](https://docs.aws.amazon.com/ko_kr/gamelift/latest/developerguide/queues-best-practices.html#queues-design-spot)

![readme/FleetScaling-001.png](readme/FleetScaling-001.png)

### Step 13.4. Fleet 생성 시 서버 프로세스 설정

위에서 진행한 프로젝트에서는 하나의 프로세스가 하나의 게임 세션을 담당합니다.

플릿 생성 시 사용했던 CLI 를 살펴봅니다.

**\ctc-game-server\scripts\GameLift\02-GameLift-CreateFleet.sh**

```bash
# https://docs.aws.amazon.com/cli/latest/reference/gamelift/create-fleet.html
# Set variables to suit your development environment
FLEET_NAME=BattleFleet
BUILD_ID=build-6eaa542c-f1bc-4c96-90ba-c6ce44b62d98
EC2_INSTANCE_TYPE=c5.large
FLEET_TYPE=SPOT
LAUNCH_PATH=BattleServer.exe
RUNTIME_CONFIGURATION='ServerProcesses=[
{LaunchPath="C:\Game\BattleServer.exe",Parameters=50001,ConcurrentExecutions=1},
{LaunchPath="C:\Game\BattleServer.exe",Parameters=50002,ConcurrentExecutions=1},
{LaunchPath="C:\Game\BattleServer.exe",Parameters=50003,ConcurrentExecutions=1},
{LaunchPath="C:\Game\BattleServer.exe",Parameters=50004,ConcurrentExecutions=1},
{LaunchPath="C:\Game\BattleServer.exe",Parameters=50005,ConcurrentExecutions=1}
]'
INSTANCE_ROLE_ARN='arn:aws:iam::831150897155:role/mzc-game-gamelift-fleet-role'

aws gamelift create-fleet \
--name "$FLEET_NAME" \
--build-id "$BUILD_ID" \
--ec2-instance-type "$EC2_INSTANCE_TYPE" \
--fleet-type "$FLEET_TYPE" \
--runtime-configuration "$RUNTIME_CONFIGURATION" \
--instance-role-arn "$INSTANCE_ROLE_ARN" \
--ec2-inbound-permissions 'FromPort=50001,ToPort=50005,IpRange=0.0.0.0/0,Protocol=TCP'
```

![readme/ServerProcesses-001.png](readme/ServerProcesses-001.png)

runtime configuration을 설정한 부분을 보면 5개의 프로세스를 설정 해 놓은 것을 볼 수 있습니다.  이렇게 설정 했을 때 실제 5개의 게임 세션까지 게임을 진행할 수 있습니다. 

만약 1000개의 프로세스를 띄운다면 Launch parameters로 넘겨준 Port 할당이 50001~51000이 될 것입니다. 

![readme/ServerProcesses-002.png](readme/ServerProcesses-002.png)

서버 구현 후 단일 인스턴스 내에서 몇 개의 프로세스를 띄우고 플레이 가능 한지 부하 테스트 진행 후 Fleet 생성 시 해당 수 만큼 ServerProcesses값을 설정 해줘야 합니다. CI/CD 구성 시 CLI 또는 SDK 를 통한 Fleet 생성 시 프로세스 수를 받아서 그만큼 프로세스를 생성하는 과정이 통합되어야 합니다. 

### Step 13.5. Fleet 인스턴스 Scaling

Fleet이 활성화되면 콘솔의 Scaling 탭에서 손쉽게 Scaling 정책을 설정할 수 있습니다. 

![readme/Scaling-001.png](readme/Scaling-001.png)

![readme/Scaling-002.png](readme/Scaling-002.png)

기본이 되는 target tracking scaling 정책 외에도 특수한 상황을 처리하기 위해 target tracking 기반 scaling을 보완하는데 쓸 수 있는 rule-based policies scaling 정책을 적용 할 수 있습니다. 

[https://docs.aws.amazon.com/gamelift/latest/developerguide/fleets-autoscaling.html](https://docs.aws.amazon.com/gamelift/latest/developerguide/fleets-autoscaling.html)

### Step 13.6. Unity로 클라이언트, 서버 통합 구현

[Unity MLAPI](https://docs-multiplayer.unity3d.com/docs/getting-started/about-mlapi)는 Unity 게임 엔진이 네트워킹을 추상화하기 위해 구축 된 중간 수준의 네트워킹 라이브러리입니다. 이를 통해 개발자는 낮은 수준의 프로토콜과 네트워킹 프레임 워크가 아닌 게임에 집중할 수 있습니다. 

Unity의 MLAPI를 이용하면 클라이언트는 조작, 서버에서 조작에 대한 계산 및 브로드캐스팅하는 방식으로 개발하는 것이 아니라 통합으로 개발 후 서버로 빌드, 클라이언트로 빌드 할 수 있습니다. 

## Step 14. 리소스 정리

1. API 게이트웨이
    1. GameAPI
2. Lambda
    1. CreateGame
    2. SearchGame
    3. Login
    4. CreatePlayer
    5. AccountJoin
    6. BattleResult
3. SQS
    1. mzc-game-sqs
4. GameLift Fleet
5. GameLift Build
6. RDS
    1. mzc-game-rds
7. RDS Subnet group
    1. mzc-game-db-subnetgroup
8. Cloud9
9. IAM
    1. 역할
        1. mzc-game-gamelift-fleet-role
        2. mzc-game-lambda-role
    2. 정책
        1. GameLiftFullAccess
10. NAT 게이트웨이
    1. mzc-game-nat
11. 인터넷 게이트웨이
    1. mzc-game-ig
12. 서브넷
    1. mzc-game-public-subnet1
    2. mzc-game-public-subnet2
    3. mzc-game-private-subnet1
    4. mzc-game-private-subnet2
13. 라우트 테이블
    1. mzc-game-private-rt
    2. mzc-game-public-rt
14. VPC
    1. mzc-game-vpc
        1. 보안 그룹이 같이 삭제 됩니다.
            1. mzc-game-lambda-sg
            2. mzc-game-rds-sg

## …

---

**END**
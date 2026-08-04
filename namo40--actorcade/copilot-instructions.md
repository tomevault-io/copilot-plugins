## actorcade

> Akka.NET + .NET 10으로 실시간 MMORPG 서버를 단계별로 만드는 교육 학습용 프로젝트입니다.

# Actorcade

Akka.NET + .NET 10으로 실시간 MMORPG 서버를 단계별로 만드는 교육 학습용 프로젝트입니다.

## 기술 스택

현재 도입된 것만 표기합니다(파트가 진행될수록 늘어납니다).

- .NET 10
- Akka.NET 1.5 (`Akka`, `Akka.Hosting`, `Akka.TestKit.Xunit2`, `Akka.Persistence`)
- MessagePack (`MessagePack` — 클라이언트↔서버 바이너리 프로토콜)
- JWT 검증 (`Microsoft.IdentityModel.JsonWebTokens` — Server/Tests 전용, Domain은 모름)
- PostgreSQL 영속화 (`Akka.Persistence.Sql.Hosting`, `Npgsql` — Server 전용, Domain은
  `Akka.Persistence` 추상만 참조하고 구체적인 DB 기술을 모름)
- `Testcontainers.PostgreSql` — Tests 전용, 통합 테스트가 자기 인프라(postgres 컨테이너)를
  스스로 기동
- Kubernetes 클러스터 (`Akka.Cluster.Hosting`, `Akka.Management`,
  `Akka.Discovery.KubernetesApi` — Server 전용, `Akka:ClusterEnabled` 스위치로 켜고 끈다)

## 솔루션 구조

```
Actorcade.slnx
├─ src/
│  ├─ Actorcade.Domain/   액터·메시지·도메인 로직
│  └─ Actorcade.Server/   ASP.NET Core 호스트 + Akka.Hosting 부트스트랩
└─ tests/
   └─ Actorcade.Tests/    Akka.TestKit 기반 테스트
```

`Actorcade.Domain`은 클러스터링(샤딩·싱글턴 등)이나 구체적인 DB 기술을 모릅니다. 도메인은
"Envelope을 어디로 보낼지"와 "무엇을 저장할지"만 인터페이스로 표현하고, 실제로 어떻게 라우팅하고
어디에 저장하는지는 상위 계층(`Actorcade.Server`, 이후 파트에서 추가될 인프라 계층)이 결정합니다.
이 경계는 Part 0부터 의도적으로 유지됩니다.

## 작업 규칙

- **파일 하나 = 타입 하나**: 클래스/레코드/인터페이스를 파일 하나에 하나씩만 정의합니다. 역할별
  폴더(`Actors/`, `Messages/`, `Protocol/`, `Persistence/` 등)로 구성합니다.
- **주석은 한글**: 이 프로젝트는 학습용이므로 "자명한 코드에는 주석을 달지 않는다"는 일반 규칙보다
  학습 기록이 우선합니다. Akka.NET 개념(슈퍼비전, PipeTo, mailbox, 샤딩 등)이 처음 등장하는
  지점에는 `[학습 노트]` 주석으로 개념을 설명합니다.
- **액터 내부 블로킹 금지**: 액터 안에서 DB 호출이나 I/O를 동기적으로 기다리지 않습니다. 비동기
  작업의 결과는 반드시 PipeTo 패턴으로 액터에게 다시 메시지로 전달합니다.
- **액터 로직은 TestKit 테스트를 동반**: 새로 추가하거나 변경한 액터 로직에는 Akka.TestKit
  기반 테스트를 함께 작성합니다.
- **main은 항상 빌드·테스트 통과 상태를 유지**합니다.
- 커밋 메시지도 한글로 작성합니다.

## 자주 쓰는 명령

```powershell
docker compose -f deploy/local/compose.yaml up -d
dotnet build Actorcade.slnx
dotnet test Actorcade.slnx
dotnet run --project src/Actorcade.Server/Actorcade.Server.csproj
```

Part 6부터는 K8s 배포도 가능합니다(로컬 `dotnet run`은 `Akka:ClusterEnabled` 기본값이
`false`라 영향받지 않습니다). 순서·확인·정리 명령은 `deploy/k8s/README.md` 참고:

```powershell
kubectl apply -f deploy/k8s/namespace.yaml
kubectl apply -f deploy/k8s/rbac.yaml
kubectl apply -f deploy/k8s/gamedb.yaml
docker build -t actorcade-server:dev .
./deploy/k8s/load-image.ps1
kubectl apply -f deploy/k8s/deployment.yaml
kubectl apply -f deploy/k8s/service.yaml
```

---
> Source: [namo40/actorcade](https://github.com/namo40/actorcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->

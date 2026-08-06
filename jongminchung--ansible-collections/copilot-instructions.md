## ansible-collections

> 이 저장소는 `dev_infra.deployment` Ansible Collection의 유일한 소스 저장소다.

# AGENTS.md

이 저장소는 `dev_infra.deployment` Ansible Collection의 유일한 소스 저장소다.

## 소비 및 릴리스 규칙

- 소비 저장소에서 로컬 경로, symlink, `main` 브랜치 또는 추가 `roles_path`로 이 Collection을 우회
  참조하지 않는다.
- 변경 순서는
  `수정 -> task check -> commit/main push -> task release:retag -> 소비 저장소 재설치 -> SHA status 검증`이다.
- `0.0.1`은 초기 개발 중에만 강제로 이동할 수 있다. 안정화 후에는 기존 태그를 변경하지 않고 새
  semantic version 태그를 발행한다.
- 비밀 값과 인벤토리는 Collection에 추가하지 않는다. Role은 소비 인벤토리가 전달한 최종 변수를
  검증하고 사용한다.

## 구조 규칙

- `k8s_deployment`은 Helm, Kubernetes manifest, wait, RKE2 image import 공통 동작만 소유한다.
- Kubernetes와 Helm API 작업은 Ansible control node에서 `delegate_to: localhost`, `become: false`,
  `run_once: true`, `ansible_playbook_python`으로 실행한다.
- `dev_infra_deployment_control_node_kubeconfig_path`는 평문 또는 whole-file Ansible Vault 경로를
  받는다. Vault 경로 자동 복호화는 local action plugin Interface이므로 Kubernetes 작업을 remote
  transport로 옮기지 않는다.
- managed node에 Collection 전용 Helm, kubectl, Python venv를 설치하지 않는다. kubectl은 control
  node 필수 의존성도 아니다.
- 서비스 Role은 manifest와 Helm values를 memory에서 전달하며 원격 artifact staging을 추가하지
  않는다.
- 플랫폼과 워크로드 Role은 public Playbook에서 FQCN과 `tasks_from`으로 직접 호출한다.
- `devinfra_*` 전달용 wrapper Role을 추가하지 않는다.
- 설치와 갱신은 `configure`에 통합하며 `install` lifecycle을 추가하지 않는다.
- public lifecycle은 `init`, `configure`, `kickstart`, `pull`, `restart`, `uninstall`이다.
- 호스트와 RKE2 `bootstrap`은 소비 저장소가 소유하며 Collection public Playbook으로 다시 제공하지
  않는다.
- public lifecycle끼리 서로 호출하지 않는다. 플랫폼과 workload 설치·갱신은 `configure`, 구성된
  stateful workload 기동은 `kickstart`가 각각 소유한다.
- `restart`는 Collection이 관리하는 platform controller와 stateful workload를 모두 재시작한다.
- `uninstall` 실행 자체를 전체 제거 승인으로 간주한다. Collection이 만든 platform, workload,
  namespace, PVC/PV, CRD와 소유가 검증된 외부 DNS record를 모두 제거하되 RKE2와 host 설정은 제거하지
  않는다.
- Collection 관리 namespace와 cluster-scoped resource는 `dev-infra-deployment` exclusive ownership을
  표시하며, uninstall은 foreign consumer를 발견하면 mutation 전에 중단한다.
- Role과 `tasks_from`은 public Playbook의 내부 구현 Interface이며 소비 저장소가 직접 호출하지 않는다.
- fresh Vault initialization은 수동 보안 경계다. Collection이 root token 또는 unseal key를 생성,
  출력, 저장하거나 inventory를 수정하지 않는다.

## 검증

변경 전에는 다음을 실행한다.

```bash
task check
task release:status
```

`release:retag`은 clean `main`과 `HEAD == origin/main`에서만 실행한다. 소비 저장소는 재설치 성공 시
기록한 receipt SHA와 원격 태그 SHA를 비교해야 하며 `.ansible/collections` 전체 삭제로 문제를
우회하지 않는다.

---
> Source: [jongminchung/ansible-collections](https://github.com/jongminchung/ansible-collections) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->

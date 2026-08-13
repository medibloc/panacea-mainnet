# Panacea v2.3.0 메인넷 업그레이드

이 가이드는 `panacea-3`에서 v2.2.0 또는 v2.2.1을 실행 중인 노드를
Cosmovisor 또는 수동 재시작 방식으로 업그레이드하는 방법을 설명합니다.
아래 명령은 Panacea 홈 디렉터리로 `$HOME/.panacea`를 사용하는 경우를
기준으로 합니다. 다른 경로를 사용한다면 해당 경로로 바꾸십시오.

## 릴리스

- 릴리스: https://github.com/medibloc/panacea-core/releases/tag/v2.3.0
- 커밋: `91c74f66aaeb0b2fc37282175eee400d0767e37f`
- AMD64 SHA256: `833c2d3c7dca815692e2050f2180a65736669c654965dae89175982c3139d499`
- ARM64 SHA256: `c2f7d6dfd97fe2091abe3d5a61eeef4930c4ca1bc5d7e3d9c8156c320283bfa6`
- 업그레이드 높이: `28,074,600`
- 예상 시각: `2026-08-20 16:00 KST` (`07:00 UTC`)
- 카운트다운: [Mintscan](https://www.mintscan.io/medibloc/block/28074600)

예상 시각과 카운트다운은 블록 생성 속도에 따라 달라질 수 있습니다. 실제
업그레이드는 지정된 블록 높이에서 실행됩니다.

## 업그레이드 개요

이번 업그레이드는 바이너리만 교체해 재시작하는 방식으로는 완료할 수
없습니다. 설정 마이그레이션이 필요하며, v2.3.0부터 애플리케이션
데이터베이스는 `goleveldb`만 지원합니다.

업그레이드 높이에 도달하기 전에 릴리스 바이너리를 검증하고 주요 파일을
백업한 다음 `app.toml`을 SDK v0.50 형식으로 마이그레이션해야 합니다. 이때
`db_backend`가 `goleveldb`인지, `app-db-backend`가 비어 있거나
`goleveldb`인지 반드시 확인하십시오. 마지막으로 검증한 바이너리를
Cosmovisor 또는 수동 재시작 방식에 맞게 준비하십시오.

## v2.3.0 주요 변경사항

- Cosmos SDK를 v0.50.15, CometBFT를 v0.38.23, IBC-Go를 v8.8.0으로
  업그레이드합니다.
- v2.3.0 업그레이드 핸들러와 Panacea NFT 모듈을 추가하고 기존 AOL 및 DID
  서명에 대한 호환성을 유지합니다.
- gRPC-Web을 API 리스너로 이동하고, 클라이언트 설정 명령을 변경하며,
  `goleveldb` 이외의 애플리케이션 DB 백엔드 지원을 제거합니다.

## 업그레이드 전

### 1. 노드 아키텍처 확인

```sh
uname -m
```

출력 결과에 맞는 절차만 진행하십시오.

| `uname -m` 출력 | 사용할 바이너리 |
| --- | --- |
| `x86_64` 또는 `amd64` | AMD64 |
| `aarch64` 또는 `arm64` | ARM64 |

### 2. v2.3.0 다운로드 및 검증

각 절차의 마지막 `cp` 명령은 이후 단계에서 공통으로 사용할 `panacead`
파일을 만듭니다.

#### AMD64

```sh
mkdir -p $HOME/.panacea/releases/v2.3.0
cd $HOME/.panacea/releases/v2.3.0

curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-amd64
curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-amd64.sha256

sha256sum -c panacead-linux-amd64.sha256

chmod +x panacead-linux-amd64
./panacead-linux-amd64 version --long | grep -E '^(version|commit):'

cp -p panacead-linux-amd64 panacead
```

체크섬 검증 결과는 `panacead-linux-amd64: OK`여야 합니다. 버전 출력에는
`version: 2.3.0`과 커밋
`91c74f66aaeb0b2fc37282175eee400d0767e37f`가 포함되어야 합니다.

#### ARM64

```sh
mkdir -p $HOME/.panacea/releases/v2.3.0
cd $HOME/.panacea/releases/v2.3.0

curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-arm64
curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-arm64.sha256

sha256sum -c panacead-linux-arm64.sha256

chmod +x panacead-linux-arm64
./panacead-linux-arm64 version --long | grep -E '^(version|commit):'

cp -p panacead-linux-arm64 panacead
```

체크섬 검증 결과는 `panacead-linux-arm64: OK`여야 합니다. 버전 출력에는
`version: 2.3.0`과 커밋
`91c74f66aaeb0b2fc37282175eee400d0767e37f`가 포함되어야 합니다.

### 3. 주요 파일 백업

설정, 검증인 키, 검증인 서명 상태를 보관할 백업 디렉터리를 만듭니다.

```sh
BACKUP_DIR="$HOME/.panacea/backups/pre-v2.3.0-files-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -m 0700 -p "$BACKUP_DIR"

tar -C $HOME/.panacea -czf "$BACKUP_DIR/config.tar.gz" config
cp -p $HOME/.panacea/data/priv_validator_state.json "$BACKUP_DIR/"
echo "$BACKUP_DIR"
```

설정 아카이브에 검증인 키가 포함됐는지 확인하고 백업 체크섬을 생성한 후
검증합니다.

```sh
tar -tzf "$BACKUP_DIR/config.tar.gz" |
  grep '^config/priv_validator_key.json$'

cd "$BACKUP_DIR"
sha256sum config.tar.gz priv_validator_state.json >SHA256SUMS
sha256sum -c SHA256SUMS
du -sh "$BACKUP_DIR"
```

키 확인 명령은 `config/priv_validator_key.json`을 출력해야 하며, 체크섬
검증 결과는 모두 `OK`여야 합니다.

### 4. `app.toml` 마이그레이션

현재 `app.toml`을 변경하기 전에 마이그레이션 결과를 별도 파일로 생성해
검토합니다.

```sh
$HOME/.panacea/releases/v2.3.0/panacead config migrate v0.50 \
  --home $HOME/.panacea \
  --stdout >$HOME/.panacea/config/app.toml.v0.50.preview

diff -u \
  $HOME/.panacea/config/app.toml \
  $HOME/.panacea/config/app.toml.v0.50.preview
```

diff에는 주로 새로운 기본 설정이 추가되고 더 이상 사용하지 않는 설정이
제거된 내용이 나타납니다. 노드별로 지정한 값이 올바르게 유지되는지
확인하십시오.

차이를 검토한 다음 실제 설정에 마이그레이션을 적용하고 권장 쿼리 제한을
설정합니다.

```sh
$HOME/.panacea/releases/v2.3.0/panacead config migrate v0.50 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app query-gas-limit 10000000 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app api.rpc-write-timeout 10 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app grpc.max-send-msg-size 10485760 \
  --home $HOME/.panacea
```

마이그레이션된 `app.toml`과 DB 백엔드 설정을 확인합니다.

```sh
$HOME/.panacea/releases/v2.3.0/panacead config view app \
  --home $HOME/.panacea >/dev/null && echo 'app.toml: OK'

grep -nE '^(db_backend|app-db-backend) =' \
  $HOME/.panacea/config/config.toml \
  $HOME/.panacea/config/app.toml
```

첫 번째 명령은 `app.toml: OK`를 출력해야 합니다. `db_backend`는
`goleveldb`여야 하며, `app-db-backend`는 비어 있거나 `goleveldb`여야
합니다. `cleveldb`, `boltdb`, `badgerdb`로 v2.3.0을 시작하면 안 됩니다.
검증에 실패하거나 다른 백엔드가 설정되어 있다면 v2.3.0을 시작하지 말고
공식 검증인 채널에 보고하십시오.

### 5. 시작 명령 확인

노드 시작 명령에 `--grpc-web.address`가 없다면 이 단계를 건너뛰십시오.
해당 옵션이 있다면 v2.3.0에서 더 이상 지원하지 않으므로 업그레이드 전에
제거해야 합니다. 옵션을 제거한 뒤에는 기존 v2.2.0 또는 v2.2.1
바이너리로 노드를 다시 시작해 정상적으로 동작하는지 확인하십시오.

### 6. 업그레이드 방식 선택

다음 중 한 가지 방식만 사용하십시오.

| 방식 | 업그레이드 높이 전 | 업그레이드 높이 도달 시 |
| --- | --- | --- |
| Cosmovisor | 업그레이드 디렉터리에 바이너리 배치 | Cosmovisor가 바이너리를 자동 전환 |
| 수동 재시작 | 검증된 릴리스 바이너리 준비 | 기존 바이너리가 중지된 후 v2.3.0 시작 |

#### Cosmovisor

검증된 바이너리를 배치하고 복사본이 동일한지 확인합니다.

```sh
readlink -f $HOME/.panacea/cosmovisor/current

mkdir -p $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin

cp -p \
  $HOME/.panacea/releases/v2.3.0/panacead \
  $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead
chmod +x $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead

cmp \
  $HOME/.panacea/releases/v2.3.0/panacead \
  $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead \
  && echo 'staged binary: OK'

$HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead version --long |
  grep -E '^(version|commit):'
```

비교 결과는 `staged binary: OK`여야 합니다. 버전은 `2.3.0`이고 위에
기재된 릴리스 커밋과 일치해야 합니다. 업그레이드 높이 전에는
`cosmovisor/current`가 현재 실행 중인 v2.2.0 또는 v2.2.1 바이너리를
가리켜야 합니다.

`cosmovisor/current`를 수동으로 변경하지 마십시오. Cosmovisor 자동 바이너리
다운로드는 비활성화된 상태로 유지하십시오.

#### 수동 재시작

검증된 바이너리를 `$HOME/.panacea/releases/v2.3.0/panacead`에
준비하십시오. 업그레이드 높이에서 프로세스 관리자가 이 바이너리를
실행하도록 전환할 준비를 하되, 그전에는 v2.3.0을 시작하지 마십시오.

## 업그레이드 높이 도달 시

**Cosmovisor:** 노드 로그를 확인하십시오. Cosmovisor가
`upgrades/v2.3.0/bin/panacead`를 선택하고 마이그레이션을 실행한 다음 블록
처리를 재개해야 합니다.

**수동 재시작:** 현재 v2.2.0 또는 v2.2.1 바이너리가 지정된 높이에서
중지될 때까지 기다리십시오. 프로세스 관리자가
`$HOME/.panacea/releases/v2.3.0/panacead`를 사용하도록 변경한 다음 노드를
시작하십시오.

v2.3.0을 시작한 후에는 마이그레이션이 끝날 때까지 기다리십시오. 오류로
종료되지 않는 한 프로세스를 중단하지 마십시오.

## 업그레이드 후

노드가 실제로 v2.3.0 바이너리로 전환됐는지 확인합니다.

- **Cosmovisor:** `readlink -f $HOME/.panacea/cosmovisor/current`가
  `upgrades/v2.3.0`을 가리켜야 합니다.
- **수동 재시작:** 프로세스 관리자가
  `$HOME/.panacea/releases/v2.3.0/panacead`를 사용해야 합니다.

버전, 적용된 업그레이드, 동기화 상태를 확인합니다.

```sh
$HOME/.panacea/releases/v2.3.0/panacead version --long |
  grep -E '^(version|commit):'

$HOME/.panacea/releases/v2.3.0/panacead query upgrade applied v2.3.0 \
  --home $HOME/.panacea \
  --node tcp://127.0.0.1:26657 \
  --output json | jq

curl -fsS --max-time 5 http://127.0.0.1:26657/status |
  jq '.result.sync_info | {latest_block_height, latest_block_time, catching_up}'
```

버전과 커밋은 릴리스와 일치해야 합니다. 또한 `catching_up`이 `false`이고
블록 높이가 계속 증가하는지 확인하십시오. 노드 로그에 마이그레이션 또는
합의 오류가 없는지도 확인해야 합니다.

## Public API 및 CLI 명령 변경사항

노드가 REST 또는 gRPC-Web을 외부에 제공하거나 `panacead` CLI로
`client.toml`을 조회하거나 변경하는 경우에만 이 섹션을 확인하십시오.
해당하지 않으면 건너뛰어도 됩니다. 아래 변경사항은 합의에 영향을 주지
않습니다.

v2.3.0에서 gRPC-Web은 더 이상 `9091` 포트의 별도 리스너를 사용하지 않고,
일반적으로 `1317` 포트를 사용하는 API 리스너를 통해 제공됩니다. 네이티브
gRPC는 `[grpc]`에 설정된 주소를 계속 사용합니다. 클라이언트가 현재 `9091`
포트를 사용한다면 API 주소로 변경하십시오. 공개 주소를 유지해야 한다면
업그레이드 높이에 리버스 프록시의 업스트림을 전환하면 됩니다. 브라우저
클라이언트가 직접 연결하는 경우에는 API 리스너 또는 프록시에 CORS를
설정하십시오.

`client.toml`을 조회하거나 변경하는 CLI 명령은 다음 형식을 사용해야
합니다.

- `panacead config get client <key>`
- `panacead config set client <key> <value>`

## 복구

체인이 재개되지 않으면 로그를 보존하고 공식 검증인 채널에 상황을 공유해
대응을 조율하십시오. 다른 검증인과 합의하지 않은 채
`--unsafe-skip-upgrades`를 사용하거나, 서명 상태를 복원하거나, 체인
데이터를 롤백하지 마십시오.

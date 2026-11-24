# 쿠웨이트박 (Team Raffle Bot)

Teams 단체 대화방에서 후보 중 원하는 수만큼 중복 없이 뽑아 주는 봇입니다.

- GitHub 저장소: [namonak/teams-raffle-bot](https://github.com/namonak/teams-raffle-bot)
- 운영 주소: `https://raffle-bot.namonak.dev`
- 구성: TypeScript, Microsoft Agents SDK, SQLite, Docker Compose (`teams-meal-bot`, `teams-poll-bot`과 같은 구조)
- 이름 규칙: GitHub 저장소만 `teams-raffle-bot`이고, 로컬 폴더·컨테이너·DB·ZIP 파일은 `team-raffle-bot`을 그대로 사용합니다.

## 사용 방법

Teams에서 `@쿠웨이트박`을 입력한 뒤, 목록에 나오는 봇 멘션을 선택해서 보내 주세요.
단체 대화방에서만 동작하며 개인 대화와 팀 채널은 지원하지 않습니다.

| 입력 | 동작 |
| --- | --- |
| `@쿠웨이트박` | 제목·후보·추첨 개수를 입력하는 카드 표시 |
| `@쿠웨이트박 도움말` | 사용법 안내 |
| `@쿠웨이트박 짜장 짬뽕 볶음밥` | 1개 추첨 |
| `@쿠웨이트박 홍길동 김철수 이영희 2명` | 2명 추첨 |
| `@쿠웨이트박 서울 부산 제주 2곳` | 2곳 추첨 |

### 직접 입력 규칙

- 후보 구분: 공백, 탭, 줄바꿈
- 추첨 개수: 마지막에 숫자만 적거나 숫자 바로 뒤에 `명`·`개`·`곳`을 붙입니다. 생략하면 1개입니다.
- 숫자와 단위는 붙여 씁니다. `2 명`처럼 띄우면 개수로 읽지 않습니다.
- 마지막 숫자는 항상 개수로 읽으므로, 숫자 후보는 `1번`처럼 적어 주세요.
- 후보 이름에 공백이 있으면 입력 카드를 사용해 주세요.

### 입력 카드

- 제목, 후보(한 줄에 하나), 추첨 개수(숫자만)를 적고 **좋아, 판 돌려!**를 누릅니다.
- 카드를 연 사람만 시작할 수 있으며, 시작 전 카드는 24시간 동안 유효합니다.
- 카드에서는 마지막 숫자도 후보 이름으로 읽습니다.
- 입력 카드가 그 자리에서 진행 카드와 결과 카드로 바뀝니다. 새 메시지는 보내지 않습니다.

### 입력 제한

- 후보 2~50개, 후보명 1~50자, 제목 1~100자, 전체 입력 3,000자 이내
- 거절 조건: 같은 후보 중복, 후보 1개, 0·음수·소수 개수, 후보 수 이상의 추첨 개수
- 중복 판단: 앞뒤 공백을 지우고 유니코드 NFC 정규화(같은 글자를 같은 코드로 맞추는 처리) 후 비교합니다. 대소문자와 중간 공백은 구분합니다.

## 추첨과 카드 갱신

- 추첨 방식: Node 표준 `crypto.randomInt`로 모든 후보에 같은 확률을 줍니다.
- 결과 확정: 시작할 때 한 번 확정해 DB에 저장합니다. 중계 문구와 하단 게이지는 연출이며 확률과 관계없습니다.
- 중계: 약 2초 간격으로 6개 장면을 보여 주고 약 12초 뒤 결과를 공개합니다. Teams와 네트워크 상황에 따라 중간 장면이 생략되거나 늦게 보일 수 있습니다.
- 동시 진행: 같은 대화방은 한 판씩, 다른 대화방은 따로 진행합니다.
- 중복 방지: 같은 요청이 다시 들어오거나 버튼을 두 번 눌러도 새 추첨을 만들지 않습니다.
- 장애 대응: 중계 갱신이 실패하면 최종 결과를 먼저 반영합니다. 429 응답(요청 한도 초과)은 Retry-After 시간만큼 기다리고, 일시 장애는 대기 시간을 늘려 가며 재시도합니다.
- 재시작: 미완료 중계는 반복하지 않고 저장된 결과로 마무리합니다.
- 재추첨 버튼은 없습니다. 새로 뽑으려면 봇을 다시 호출해 주세요.

## 로컬 개발

준비물: Node 24 이상, npm, bash, zip, unzip

```bash
npm ci
cp .env.example .env    # 쿠웨이트박 전용 앱 정보를 입력해 주세요
npm test
npm run typecheck
npm run build
node --env-file=.env --import tsx src/index.ts    # 소스로 실행
node --env-file=.env dist/index.js                # 빌드 결과로 실행
```

- `npm start`는 `.env`를 읽지 않고 셸 환경 변수를 사용합니다.
- 인증 필수값이 없으면 서버가 시작되지 않습니다.
- 테스트 범위: 입력·추첨 경계값, 카드 소유권, 중복·동시 요청, DB 재개방, 갱신 실패와 재시작, 인증, ZIP·아이콘. 실제 Teams 인증과 화면 표시는 확인하지 않습니다.

| 환경 변수 | 값 |
| --- | --- |
| `MicrosoftAppId` | Azure 봇 ID (Developer Portal의 앱 ID가 아닙니다) |
| `MicrosoftAppPassword` | 봇 클라이언트 비밀값 |
| `MicrosoftAppTenantId` | Microsoft 365 테넌트 ID |
| `PUBLIC_BASE_URL` | `https://raffle-bot.namonak.dev` (기록용이며 서버 바인딩이나 프록시를 설정하지 않습니다) |
| `DATABASE_PATH` | 로컬 `./data/team-raffle-bot.sqlite`, 컨테이너 `/data/team-raffle-bot.sqlite` |
| `PORT` | 컨테이너 내부 `3978` |

## 보안 검사

`.github/workflows/security.yml`이 main 푸시, PR, 수동 실행, 매일 02:50 KST에 실행됩니다.

- 의존성 검사: 테스트와 타입 검사 뒤 `npm audit --omit=dev --audit-level=moderate` 실행
- 이미지 검사: Docker 이미지를 빌드해 Trivy(컨테이너 이미지 취약점 검사 도구) v0.76.2로 수정 가능한 HIGH·CRITICAL 취약점 검사
- 차단 기준: 두 검사 중 하나라도 취약점을 찾으면 워크플로가 실패합니다.
- 버전 고정: 간접 의존성 `body-parser` 2.4.0, `proxy-addr` 2.0.8을 `package.json`의 `overrides`로 고정합니다.
- 운영 이미지: 실행에 쓰지 않는 npm과 npx를 삭제합니다. 컨테이너 안에서는 `node`만 사용해 주세요.
- Dependabot: GitHub 취약점 알림과 자동 보안 업데이트를 켜 두었습니다.

## 서버 배포

1. 저장소를 내려받고 `.env`를 설정합니다. 다른 봇의 앱 ID와 비밀값은 재사용하지 않습니다.

   ```bash
   git clone https://github.com/namonak/teams-raffle-bot.git ~/workspace/teams-raffle-bot
   cd ~/workspace/teams-raffle-bot
   cp .env.example .env
   chmod 600 .env
   ```

2. Caddyfile에 아래 설정을 추가하고, `caddy validate`로 검사한 뒤 `caddy reload`로 재시작 없이 반영합니다. HTTPS 인증서는 Caddy가 자동으로 발급하고 갱신합니다.

   ```caddyfile
   raffle-bot.namonak.dev {
       reverse_proxy 127.0.0.1:3980
   }
   ```

3. `docker compose up -d --build`로 실행합니다. 구형 환경에서는 `docker-compose`를 사용합니다.
4. `curl https://raffle-bot.namonak.dev/healthz`가 `{"ok":true}`를 반환하는지 확인합니다.

운영 기준

- 호스트 포트: 3980 (식사 봇 3978, 투표곰 3979와 겹치지 않음)
- 자동 실행: Compose의 `restart: unless-stopped`로 서버를 재부팅해도 다시 실행됩니다. systemd로 관리하려면 `deploy/team-raffle-bot.service`의 WorkingDirectory를 서버 경로로 바꿔서 사용해 주세요.
- 데이터: `raffle-data` 볼륨에 보관합니다. `docker compose down -v`는 볼륨을 삭제하므로 사용하지 않습니다.
- 실행 단위: 컨테이너 하나로만 실행합니다. 같은 DB를 여러 봇 인스턴스에 연결하지 않습니다.
- 도메인 변경 시 함께 바꿀 항목: `.env`, DNS·Caddy, Azure 봇 메시지 엔드포인트, `appPackage/manifest.json`의 `validDomains`

코드 업데이트 반영

```bash
cd ~/workspace/teams-raffle-bot
git pull --ff-only
docker compose up -d --build
```

서버 코드만 바뀐 경우에는 Teams 앱을 다시 설치하지 않아도 됩니다.

## Teams 앱 등록과 설치

- 등록: Teams Developer Portal과 Azure Bot에서 쿠웨이트박 앱을 등록하고 Teams 채널을 연결합니다.
- 메시지 엔드포인트: `https://raffle-bot.namonak.dev/api/messages`
- 서버 설정과 일치 필요: 봇 ID, 비밀값, 테넌트
- 조직에서 사용자 지정 앱 업로드를 허용해야 합니다.

ZIP 만들기(프로젝트 최상단에서 실행)

```bash
bash scripts/package-teams-app.sh
```

- 출력 파일: `dist/team-raffle-bot-YYYYMMDD-HHMMSS.zip` (한국 시간 기준)
- 포함 파일: `manifest.json`, `color.png`(192×192), `outline.png`(32×32, 투명 배경). 비밀값은 들어가지 않습니다.
- 앱 ID: 환경 변수나 `.env`의 `MicrosoftAppId`(UUID 형식)로 manifest를 채웁니다. `.env`를 셸 코드로 실행하지 않습니다.
- 현재 manifest 버전: 1.0.1

설치와 업데이트

- 처음 설치: Teams 앱 관리 화면에서 **사용자 지정 앱 업로드**로 ZIP을 올린 뒤 단체 대화방에 추가해 주세요.
- 업데이트: 이름·아이콘·manifest를 바꾸면 manifest의 `version`을 올리고, 새 ZIP을 같은 방법으로 업로드해 주세요. 앱을 삭제하고 다시 설치할 필요는 없습니다.
- 주의: ZIP으로 설치한 앱은 Developer Portal에서 버전을 올려도 업데이트되지 않습니다.

## 장애 확인과 복구

```bash
docker compose logs --tail=100 bot
```

| 로그 | 의미 |
| --- | --- |
| `추첨 카드 갱신: ID retry` | 저장된 결과를 자동으로 다시 반영하는 중 |
| `attention` | 카드 삭제나 권한 오류처럼 운영자 확인이 필요한 상태 |
| `입력 카드 게시 확인 필요`, `추첨 카드 게시 확인 필요` | 최초 게시 응답이 유실되었을 수 있는 상태. 카드 중복을 막기 위해 자동 재게시하지 않고, 그 대화방의 새 추첨도 막습니다. 확정 결과는 DB에 남아 있습니다. |

복구 절차는 봇을 멈추고 대화방의 실제 게시 상태를 확인한 뒤에만 진행해 주세요.

1. 봇을 멈춥니다: `docker compose stop bot`
2. Teams 메시지의 링크 복사나 운영 도구로 메시지 ID를 확인합니다.
3. 아래 명령 중 상황에 맞는 하나를 실행합니다. `종류`는 `draft`(입력 카드) 또는 `raffle`(추첨 카드)입니다.
4. 봇을 다시 시작합니다: `docker compose start bot`

카드가 게시된 경우: 입력 카드는 준비 상태로 연결하고, 추첨은 저장된 당첨자를 유지한 채 최종 결과 반영을 예약합니다.

```bash
docker compose run --rm -T --no-deps bot node --input-type=module - 'raffle' '작업 ID' 'Teams 메시지 ID' <<'NODE'
import { DatabaseSync } from 'node:sqlite';
const [kind,id,messageId]=process.argv.slice(2);
if(!['draft','raffle'].includes(kind)||!id||!messageId)throw new Error('종류와 ID를 확인하세요');
const db=new DatabaseSync(process.env.DATABASE_PATH);
const status=kind==='draft'?'ready':'retry';
const result=db.prepare(`UPDATE ${kind} SET status=?, data=json_set(data,
  '$.status', ?, '$.messageId', ?, '$.stage', 6, '$.nextAt', 0)
  WHERE id=? AND status IN ('uncertain','attention')`).run(status,status,messageId,id);
if(result.changes!==1)throw new Error('복구할 작업이 없거나 상태가 다릅니다');
db.close();
NODE
```

카드가 게시되지 않았거나 삭제된 경우: 상태만 취소합니다. 후보와 당첨자는 지우지 않으며 재추첨하지 않습니다.

```bash
docker compose run --rm -T --no-deps bot node --input-type=module - 'raffle' '작업 ID' <<'NODE'
import { DatabaseSync } from 'node:sqlite';
const [kind,id]=process.argv.slice(2);
if(!['draft','raffle'].includes(kind)||!id)throw new Error('종류와 ID를 확인하세요');
const db=new DatabaseSync(process.env.DATABASE_PATH);
const result=db.prepare(`UPDATE ${kind} SET status='cancelled',
  data=json_set(data,'$.status','cancelled') WHERE id=? AND status IN ('uncertain','attention')`).run(id);
if(result.changes!==1)throw new Error('취소할 작업이 없거나 상태가 다릅니다');
db.close();
NODE
```

## 백업

- DB에는 후보와 사용자·대화방 식별자가 들어 있습니다. 데이터와 백업 파일의 접근 권한을 제한해 주세요.
- 기록은 자동으로 삭제하지 않습니다.

```bash
mkdir -p backups
docker compose stop bot
docker cp team-raffle-bot:/data/team-raffle-bot.sqlite "backups/team-raffle-bot-$(date +%Y%m%d-%H%M%S).sqlite"
docker compose start bot
```

복원할 때는 봇을 멈추고 현재 DB를 따로 보관한 뒤 백업 파일을 복사해 주세요.

## Teams 실사용 확인 항목

- 두 계정으로 입력 카드 소유권, 만료·오류 안내, 수정 중 입력 유지
- 텍스트·카드 입력의 단일·복수 추첨, 빠른 이중 클릭, 같은 대화방 동시 요청
- 시작 후 원래 카드만 바뀌고 결과까지 메시지가 추가되지 않는지
- 데스크톱·모바일에서 명령 제안, 줄바꿈, 중계와 최종 결과 표시
- 서버 재시작 후 확정 결과 유지와 같은 카드 복구

## 참고 문서

- [설계 문서](docs/superpowers/specs/2026-10-15-team-raffle-bot-design.md)
- [구현 계획](docs/superpowers/plans/2026-10-15-team-raffle-bot.md)

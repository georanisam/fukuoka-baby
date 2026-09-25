# 후쿠오카 아기랑 — 작업 인수인계

새 세션은 이 파일부터 읽는다.

## 개요

2026-10-06(화)~10-09(금) 후쿠오카 가족여행(아기 18개월 + 성인 2인)용 개인 웹앱. 아이폰 홈 화면에 추가해서
앱처럼 쓰고, **서버·DB 없이 브라우저 localStorage에만 저장**한다. 여행 중 해외 데이터가 불안정할 수 있어
오프라인에서도 열리는 게 핵심(서비스워커).

사용자는 중학교 교사이고 개발 경험 없이 바이브코딩으로 진행 중. 코드를 설명하기보다 고치고 검증하고
배포까지 끝내는 것이 작업 패턴.

## 위치·배포

- **작업(빌드·git) 폴더**: `C:\Users\user\Desktop\hackthon\fukuoka-baby` — H: 드라이브(구글 드라이브 동기화)는
  느려서 git 작업은 반드시 C:에서 한다. H: 폴더(`H:\다른 컴퓨터\2026 데스크탑\2026 전일중학교\바이브코딩\fukuoka-baby`)는
  사본이다. 미러 명령:
  `robocopy "C:\Users\user\Desktop\hackthon\fukuoka-baby" "H:\다른 컴퓨터\2026 데스크탑\2026 전일중학교\바이브코딩\fukuoka-baby" /MIR /XD .git .vercel`
- **GitHub**: https://github.com/georanisam/fukuoka-baby (private), `main` 브랜치.
- **Vercel**: 프로젝트 `fukuoka-baby` (georanisam-7890 계정). 배포 흐름: git commit → `git push` → Vercel 자동 배포.
  CLI 직접 배포(`vercel --prod`)는 하지 않는다.
- git 커밋 아이덴티티는 repo-local `georanisam <georanisam@users.noreply.github.com>`.
- **도메인(주소)을 바꾸면 사용자 폰의 저장 데이터가 새로 시작된다**(localStorage는 주소별). 프로젝트/도메인 이름 변경 금지.

## 구조 (빌드 없는 정적 사이트)

| 파일 | 역할 |
|---|---|
| `index.html` | 앱 전체(HTML+CSS+JS 한 파일) |
| `manifest.json` | 홈 화면 앱 설정 |
| `sw.js` | 오프라인 캐시(stale-while-revalidate). **`CACHE` 이름의 버전 숫자(v1→v2)를 올리면 폰이 옛 캐시를 버린다** |
| `icon-192/512.png`, `apple-touch-icon.png` | 앱 아이콘(아기 얼굴 + 종이비행기) |

## 저장 데이터 (localStorage 키)

`packing-data`(준비물) · `itinerary-data`(일정) · `expenses-list`(경비, 항목마다 `day` 일차) ·
`resto-data`(대체식당) · `shopping-data`(쇼핑) · `jp-places`(일본어 장소 카드) · `fx-rate`(환율, 기본 9.3) ·
`meta-last-backup`(마지막 백업 시각, 백업 파일에는 미포함).
**백업 파일**(`fukuoka-baby-backup` v1)에는 위 중 meta 제외 전부가 들어간다. 데이터 항목을 추가하면
`buildBackup` / `parseBackup` / `applyBackupText` / `saveAll`을 같이 고칠 것(옛 백업에 없는 필드는 선택 처리).

## 핵심 구현 메모

1. **XSS 방지**: 사용자 입력은 innerHTML에 넣기 전에 반드시 `esc()`를 거친다(따옴표 들어간 장소명이 화면을
   깨뜨리던 원본 버그 수정). 새 렌더 코드에도 지킬 것.
2. **오늘 일정 자동 표시**: `TRIP_START`(2026-10-06) 기준으로 여행 기간에 열면 오늘 일차의 일정 탭이 열림.
3. **일정 순서 바꾸기**: 카드별 ▲▼. `dist`(이동 거리 칩)는 항목에 붙어 있어 같이 이동한다.
4. **홈 화면 앱 백업**: iOS 홈 화면 앱에서는 `<a download>`가 앱 화면을 덮어버릴 수 있어, 공유시트(`navigator.share`)
   → 실패하면 클립보드 복사로 폴백한다.
5. **일본어 카드**: 확실한 이름만 기본 제공(주소 오기 방지). 호텔 등은 사용자가 앱에서 직접 추가.
   "크게 보여주기" 오버레이는 택시·직원에게 화면을 보여주는 용도.
6. **iOS 저장 주의**: 홈 화면 앱과 Safari는 저장 공간이 따로다. 홈 화면 아이콘을 삭제하면 데이터도 삭제된다.

## 현재 상태 (2026-09-25)

기능 완료·배포(https://fukuoka-baby.vercel.app). 배포 주소에서 서비스워커 등록·캐시 6개 파일까지 확인함.
**아직 실기기(아이폰) 검증 전**: 비행기 모드로 열기, 공유시트 백업 저장, 홈 화면 아이콘 표시.

## 다음 후보 (선택)

아기 기록(분유·기저귀·낮잠), 백업 안 한 지 N일 경과 알림 배너, 귀국용 준비물 리스트.

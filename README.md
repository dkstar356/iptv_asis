# IPTV_ASIS — GitHub Pages 게시용

광고를 편성하기 전 원본 IPTV_ASIS입니다. 기존 화면과 메뉴·리모컨 조작을 유지하고 내장 이미지만 외부 파일로 분리했습니다. 이미지 데이터는 재압축하지 않았습니다. 원본에는 별도 영상 파일이 없습니다.

## 업로드

1. ZIP을 압축 해제합니다.
2. index.html, .nojekyll, README.md와 assets 폴더를 저장소 최상위에 함께 업로드합니다. ZIP 파일 자체를 올리는 방식으로는 게시되지 않습니다.
3. 이미지는 반드시 assets/images/ 아래에 원래 파일명으로 업로드합니다. 개별 파일 업로드도 가능하지만 이 폴더 경로를 유지해야 합니다.
4. 폴더 생성을 위해 빈 이미지 파일을 만들지 마세요. 필요하면 assets/images/keep.txt를 만든 후 실제 이미지 파일을 업로드하세요.
5. 브라우저에서는 파일을 100개 이하씩 나눠 업로드할 수 있습니다. GitHub Desktop에서 전체 폴더를 복사해 Commit/Push하는 방법도 가능합니다.
6. Settings → Pages → Deploy from a branch에서 업로드한 브랜치(예: main)와 /(root)를 선택합니다.
7. 배포 완료 후 게시 URL을 엽니다. 이전 화면이 보이면 Ctrl+F5로 새로고침합니다.

index.html만 옮기면 이미지가 표시되지 않습니다. assets/images 폴더와 함께 유지해 주세요.

## AI 테스트용 최적화 버전

- 전체 299개 이미지에 시작부터 src를 지정하던 구조를 변경했습니다. 시작 시 첫 화면 1개를 먼저 로드하고, 준비 후 현재 위치의 인접 화면을 최대 4개까지 순차적으로 미리 로드합니다.
- 화면 이미지의 src를 유지하는 캐시는 최대 6개입니다. 로딩 도중에는 일시적으로 추가 이미지가 있을 수 있고, 브라우저 자체의 네트워크/메모리 캐시는 별도로 관리됩니다.
- 사용하지 않는 이미지의 src 참조를 해제합니다. 화면 원본 이미지와 메뉴 이동 규칙은 유지합니다.
- 이미지 로드와 디코딩이 끝나고 화면 표시 기회를 거친 후 준비 신호를 제공합니다. 로딩 중 키 입력은 순서대로 대기열에서 처리합니다.
- 오류 발생 시 준비 상태를 false로 유지하며 IPTVTest.state.error로 이유를 제공합니다. 네트워크 복구 후 IPTVTest.retry()로 다시 시도할 수 있습니다.

### 자동 평가 도구 연동

Playwright를 사용한다면 최초 진입과 각 방향키 입력 뒤에 아래처럼 준비 완료를 기다립니다. document의 load 이벤트나 고정된 대기 시간만으로 화면 준비를 판단하지 마세요.

```js
await page.goto(URL, {waitUntil: 'domcontentloaded'});
await page.waitForFunction(() => Boolean(window.IPTVTest));
await page.evaluate(() => window.IPTVTest.whenReady(30000));
await page.locator('#tv').focus();
await page.keyboard.press('ArrowDown');
await page.evaluate(() => window.IPTVTest.whenReady(30000));
console.log(await page.evaluate(() => window.IPTVTest.state));
```

IPTVTest.state에는 ready, busy, asset(표시 완료 화면), requested(이동 대상), queued, resident, error가 포함됩니다. #tv의 data-ready 속성은 준비 상태를, data-screen은 표시 완료 화면의 파일명을 나타냅니다. iptv:loading, iptv:ready, iptv:state, iptv:error 이벤트도 제공합니다. 자동화에서는 개별 ready 이벤트만 구독하기보다 whenReady() 또는 IPTVTest.ready를 사용하면 대기 중 키 입력까지 처리 완료했는지 확인할 수 있습니다.

기존 GitHub 저장소의 assets/images가 동일한 IPTV_ASIS 패키지라면 index.html만 교체해도 됩니다. 볼보 광고 편성본과 혼합하지 마세요. 이 패키지는 광고 편성 전 IPTV_ASIS 원본 기준입니다.

### 검증 범위

이미지 299개 데이터와 경로, JavaScript 문법을 확인했습니다. 브라우저 DOM과 이미지 로딩을 모사한 실행에서 원본과 352개 이동 결과를 비교하고, 연속 키 입력·캐시 제한·이미지 오류 후 재시도를 검사했습니다. 실제 디지털트윈 환경에서의 동시성 10 안정성이나 실제 브라우저의 메모리 절감량은 아직 측정하지 않았습니다. 첫 비교 평가는 동시성 1에서 시작하고 같은 시나리오로 점차 늘려 주세요.

전체 이미지 원본이 포함되므로 ZIP 크기는 이전과 비슷합니다. 최적화 대상은 초기 로딩량과 실행 중 이미지 참조 수입니다.

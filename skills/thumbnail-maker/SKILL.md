---
name: thumbnail-maker
description: 'Use when making video thumbnails — YouTube longform (1280×720), Shorts/Reels covers (1080×1920) — always as a multi-variant set for A/B testing. Image generation via the Higgsfield CLI (default model gpt_image_2). Triggers: "썸네일 만들어줘", "썸네일 뽑아줘", "유튜브 썸네일", "커버 이미지", "make thumbnails", "CTR 썸네일". 일반 마케팅 이미지는 image-gen.'
---

# Thumbnail Maker

썸네일은 CTR 장치다 — 예쁜 이미지가 아니라 **클릭 근거**. 항상 **변형 세트
(기본 4개)** 로 뽑아 A/B 테스트에 넘긴다. 생성 정책은 image-gen 스킬과 동일:
**모든 생성은 Higgsfield CLI 경유, 기본 모델 gpt_image_2** — 다른 경로로
말없이 전환 금지 (실패 = 보고).

## Hard rules

1. **1장 금지, 항상 4개+ 변형.** 최소 3개는 같은 영상용 경쟁 후보 — YouTube
   "Test & compare"(썸네일 3개 A/B)에 바로 걸 수 있게.
2. **문구 = 영상의 첫 주장(훅)과 일치.** 어긋나면 이탈률로 돌아온다.
   ≤12자(KR) / 3–5단어(EN), 2줄 이내, **왼쪽 배치** (모바일 우하단은 재생시간
   뱃지가 가린다). 숫자/스테이크 하나는 보이게 ($20, 99%). 낚시 금지.
3. **문구까지 통생성이 기본** — 완성 썸네일(배경+인물+텍스트)을 gpt_image_2
   원샷으로. 프롬프트는 **줄 단위로 정확히**: `first line reads exactly "…"
   in <color>, second line reads exactly "…"` + "render every character
   EXACTLY". 생성 후 **글자 단위 READ 검증 필수** — 흔한 실패는 기호 누락
   ("$20"→"0"). 오탈자 = 같은 모델로 재생성(문구를 짧게 단순화해도 됨),
   **인페인트 금지, 오탈자 배송 금지.** HTML/PIL 오버레이는 통생성이 반복
   실패할 때만 쓰는 최후 폴백 (유저 보고 후; 스크림은 하드 사각형 금지,
   투명으로 빠지는 방향성 그라디언트만).
4. **실제 얼굴 = 누끼 레퍼런스 통생성.** 유저 제공 사진만 사용, 얼굴 지어내기
   금지. 사진을 그대로 넣지 말고: rembg(`birefnet-portrait`) 누끼 → 박힌
   자막/캡션 띠는 누끼 전에 제거(해당 영역 알파 0 → 알파 bbox 크롭) → 흰 배경에
   콜라주한 **한 장**을 `--image` 레퍼런스로 주고 "preserve the EXACT faces"로
   통생성. 2인 이상은 좌우 배치 한 장으로 합쳐 LEFT/RIGHT를 프롬프트에서 지명.
   머리/옷을 바꿔야 하면 **레퍼런스를 먼저 편집** (`--image` + "keep exact
   face, change ONLY the hair to …") 후 그걸로 생성. PIL로 누끼를 그라디언트
   배경에 합성하는 방식은 금지 — 룩이 싸구려로 갈라진다.
5. **변형 중 최소 1개는 타깃 채널의 실제 피드 문법.** 생성 전에 그 채널(없으면
   같은 장르 상위 채널) 썸네일 3–5개를 보고 문법을 추출 — 색/하이라이트 박스/
   말풍선/구도 — 그대로 카피한 변형을 만든다. 나머지는 드라마틱 high-CTR 계열
   (네온, 과장 표정). 4장이 전부 같은 룩이면 A/B가 아니라 같은 카드 4장이다.
6. **검증은 눈으로, 실제 노출 크기로**: 각 변형을 **168×94(유튜브 목록 크기)로
   축소해 READ** — 문구 읽히고 표정 보이면 합격. 얼굴은 레퍼런스 사진과 대조,
   왜곡이면 재생성. 데스크톱 크기로만 보고 통과시키지 않는다.
7. 셋업/명령/JSON 파싱/다운로드 스니펫은 image-gen 스킬 —
   `higgsfield account status`로 계정·크레딧 확인 후 생성.

## 규격

| 대상 | 크기 | 비고 |
|---|---|---|
| YouTube 롱폼 | 1280×720 (16:9), <2MB | `--aspect_ratio 16:9`로 생성 → 1280×720 리사이즈, 2MB 초과 시 JPEG q90. 업로드는 organic-social의 mediaItems[].thumbnail |
| Shorts/Reels 커버 | 1080×1920 (9:16) | 첫 프레임 룰 우선 — 커버는 IG 그리드 1:1 센터 크롭 안전영역(y 420–1500) 고려 |
| 커뮤니티/배너 | 용도별 | image-gen으로 라우팅 |

## Workflow

1. **재료 수집**: 영상의 훅/첫 대사(트랜스크립트 있으면 그대로), 얼굴 사진
   (→ 룰 4대로 누끼 레퍼런스 제작), 브랜드 토큰(DESIGN.md), **타깃 채널 썸네일
   3–5개 관찰** (룰 5의 문법 추출).
2. **변형 축 설계 (4개 기본)**: ① 채널 문법 카피 ② 인물 리액션 + 큰 문구
   ③ 숫자 훅 중심 타이포 ④ 결과물/비포애프터 또는 호기심 갭. 축이 겹치는
   4장은 A/B가 아니다.
3. **통생성**: 변형별로 구도 + 줄별 문구·색 + "render every character EXACTLY"
   + 누끼 레퍼런스를 한 프롬프트에 넣어
   `higgsfield generate create gpt_image_2 --image ref.png --aspect_ratio 16:9
   --quality high --resolution 2k --wait --json`. 한국어 문구는 의미 단위
   줄나눔을 프롬프트에 명시.
4. **QA (순서대로)**: 글자 단위 READ → 168×94 축소 READ → 얼굴 레퍼런스 대조 →
   리사이즈·파일 크기. 하나라도 불합격이면 그 변형만 재생성.
5. **딜리버리**: `thumb_A~D.png` + 각 변형의 축 설명 한 줄 + 추천 1순위와 이유.
   YouTube면 Test & compare에 3개 걸라고 안내; CTR 결과로 다음 세트를 학습.

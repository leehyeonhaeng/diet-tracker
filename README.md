# 🥗 Diet Tracker

개인 식단 기록 및 영양소 관리 웹앱. 구글 로그인으로 어디서든 데이터를 동기화할 수 있다.

**[→ 앱 바로가기](https://leehyeonhaeng.github.io/diet-tracker/diet-tracker.html)**

---

## 📌 만들게 된 배경

헬스 + 러닝 병행하면서 다이어트 식단을 관리할 필요가 생겼다. 기존에 노션으로 기록하던 걸 더 편하게 쓰고 싶었고, 주변 사람들도 같이 쓸 수 있도록 멀티유저를 지원하는 웹앱으로 만들었다.

목표: **키 180cm, 85kg → 75kg** (월 1.5~2kg 감량 페이스)

---

## ✨ 주요 기능

- **구글 로그인** — 계정별 데이터 분리, 기기 간 동기화
- **식단 기록** — 날짜별 아침/점심/저녁 식사 입력
- **음식 자동완성 + 칼로리 자동계산** — 음식명 입력 시 자동완성, g 입력하면 kcal 자동 계산
- **내 음식 DB** — 자주 먹는 음식 직접 추가 (100g 기준 영양소 저장)
- **식단 템플릿** — 러닝 기간 / 헬스 복귀 후 두 가지 모드
- **영양소 달성률** — 칼로리/단백질/탄수화물/지방 목표 대비 프로그레스 바
- **히스토리** — 날짜별 기록 조회

---

## 🎯 식단 목표

| 시기 | 목표 칼로리 | 단백질 | 탄수화물 | 지방 |
|------|------------|--------|----------|------|
| 러닝 기간 | 1,523 kcal | 152g | 121g | 43g |
| 헬스 복귀 후 | 1,733 kcal | 154g | 153g | 43g |

TDEE 대비 -600~800kcal 적자 유지. 단백질 체중(kg) × 1.7g 이상 확보.

---

## 🏗 아키텍처

```
[사용자 브라우저]
      │
      ▼
[GitHub Pages]          ← 정적 HTML/CSS/JS 호스팅
      │
      ├─ [Supabase Auth] ← 구글 OAuth 로그인
      │
      └─ [Supabase DB]   ← PostgreSQL (diet_records 테이블)
```

### 데이터 흐름

1. 사용자가 GitHub Pages URL 접속
2. Supabase Auth로 구글 로그인
3. 식단 입력 → Supabase DB에 upsert
4. 히스토리/요약 조회 → Supabase DB에서 fetch

---

## 🛠 기술 스택

| 영역 | 기술 | 선택 이유 |
|------|------|----------|
| 프론트엔드 | Vanilla HTML/CSS/JS | 빌드 없이 바로 배포 가능, 단순한 구조 |
| 호스팅 | GitHub Pages | 무료, 깃헙 레포 연동으로 배포 간단 |
| 인증 | Supabase Auth + Google OAuth | 무료, 설정 간단, 멀티유저 지원 |
| DB | Supabase (PostgreSQL) | 무료, RLS로 유저별 데이터 격리, REST API 제공 |
| 음식 DB | 로컬스토리지 (브라우저) | 개인 커스텀 음식은 기기 로컬 저장 |

### Supabase를 선택한 이유

- Firebase 대비 SQL 기반이라 데이터 구조가 명확
- RLS(Row Level Security)로 유저별 데이터 격리가 간단
- 무료 플랜으로 소규모 사용 충분
- REST API 자동 생성으로 별도 백엔드 불필요

---

## 🗄 DB 스키마

```sql
create table diet_records (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  date date not null,
  mode text not null,           -- 'running' | 'gym'
  meals jsonb not null,         -- { 아침: [...], 점심: [...], 저녁: [...] }
  memo text default '',
  created_at timestamptz default now(),
  unique(user_id, date)
);

-- RLS: 본인 데이터만 접근 가능
alter table diet_records enable row level security;
create policy "본인 데이터만 접근" on diet_records
  for all using (auth.uid() = user_id);
```

### meals JSONB 구조

```json
{
  "아침": [
    { "name": "그릭요거트", "g": 100, "kcal": 59, "p": 10, "c": 4, "f": 0.4 }
  ],
  "점심": [...],
  "저녁": [...]
}
```

---

## 🚀 배포 방법

1. 이 레포 fork
2. `diet-tracker.html` 에서 Supabase URL/anon key 본인 것으로 교체
3. GitHub Pages 활성화 (Settings → Pages → main branch)
4. Supabase Auth URL Configuration에 GitHub Pages URL 등록

---

## 📅 개발 일지

[DEVLOG.md](./DEVLOG.md) 참고

---

## 📋 앞으로 추가할 것

- [ ] 체중 기록 및 그래프
- [ ] 주간/월간 영양소 통계
- [ ] 음식 DB 클라우드 동기화 (현재는 브라우저 로컬)
- [ ] 운동 기록 연동

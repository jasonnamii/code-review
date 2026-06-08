# 수정 디폴트 카탈로그 — 결함별 권장 처방

각 결함에 붙일 **표준 처방의 근거**다. 베끼지 말고 코드 맥락에 맞게 번역한다. 처방엔 [신뢰도] + 가능하면 최소 패치(diff). 저자 결정 존중 — 권장안 + 대안(절대규칙 7).

> 처방 원칙: ① 최소 변경으로 결함만 제거 ② 같은 결함이 재발 안 하게(구조적) ③ 테스트로 고정 가능하게 ④ 패치는 적용 가능한 형태로.

---

## 정확성

| 결함 | 권장 처방 | 대안 |
|---|---|---|
| null 전파 | 진입점에서 검증·early return, 옵셔널 타입 | 디폴트값·null object |
| off-by-one | 경계 명시·반열림 구간 일관 | 표준 이터레이터/슬라이스 |
| 부동소수 | 정수(최소단위)·decimal 타입 | epsilon 비교(맥락 한정) |
| 가변 디폴트 | None 디폴트 후 내부 생성 | 불변 타입 |

## 상태·동시성

| 결함 | 권장 처방 | 대안 |
|---|---|---|
| check-then-act 레이스 | 원자적 연산(`putIfAbsent`·조건부 UPDATE)·DB 유니크 | 락(범위 최소화) |
| 멱등성 부재 | 멱등키 + 유니크 제약 + 결과 캐시 | 자연키 dedup |
| 부분 실패 | 트랜잭션, 안 되면 보상(saga)·아웃박스 | 재시도 + 멱등 |
| 캐시 스탬피드 | single-flight·`getOrLoad` | 짧은 락·확률적 조기 갱신 |
| 캐시 무효화 | write 시 무효화·짧은 TTL | write-through |

## 에러·자원

| 결함 | 권장 처방 |
|---|---|
| 자원 누수 | `finally`/`defer`/`with`/RAII/`use`로 보장, cleanup |
| 삼킨 예외 | 다시 던지거나 Result로 신호, 맥락 로깅 |
| 타임아웃 없음 | 모든 외부 호출에 타임아웃 + 서킷브레이커 |
| 재시도 | 지수 백오프 + 지터, 멱등 대상만, 상한 |

## 계약·마이그레이션

| 결함 | 권장 처방 |
|---|---|
| 하위호환 깸 | 필드 추가형(제거 ✗)·버전·디폴트, deprecate→제거 단계 |
| 마이그레이션 비가역 | 역마이그레이션, 확장→백필→축소(무중단) |
| 반환 계약 불일치 | 한 형태로 통일(throw or Result), 문서화 |

## 보안 (→ `security-detector.md`)

| 결함 | 권장 처방 |
|---|---|
| 인젝션 | 파라미터 바인딩·allowlist·이스케이프 |
| authz | 서버 매 호출 검증, 소유자 확인, 없음=404 |
| 비밀 노출 | 시크릿 매니저, 커밋됐으면 로테이션 |
| 약한 암호 | bcrypt/argon2, CSPRNG, TLS |
| (PII·라이선스 적법성) | 처방 ✗ → 라우팅 |

## 성능

| 결함 | 권장 처방 |
|---|---|
| N+1 | 배치·조인·프리페치·DataLoader |
| O(n²) | 해시·정렬·인덱스로 강등 |
| 루프 내 할당 | 루프 밖 추출·재사용·스트리밍 |
| 메모리 누수 | 바운드 캐시(LRU)·구독 해제 |

## 파트별 (→ `part-lens.md`)

| 파트 | 대표 처방 |
|---|---|
| 프론트 | `useEffect` cleanup·의존성 정정·메모·AbortController·출력 이스케이프 |
| Android | `viewModelScope`·`ViewModel` 상태보존·`Dispatchers.IO`·약참조 Context |
| iOS | `[weak self]`·`guard let`·`@MainActor`·`AnyCancellable` 보관 |
| AI/ML | split 후 전처리·시드 고정·NaN 가드·입력/프롬프트 분리·출력 검증 |

---

## 처방 출력 예
```
🔧 cache.ts:42 — 동시 미스로 DB 중복 조회
권장: single-flight로 동시 미스 1회 합침 [신뢰도: 상 — 동시 진입 확인됨]
```diff
- const u = await db.getUser(id); cache.set(id, u)
+ const u = await cache.getOrLoad(id, () => db.getUser(id))
```
대안: 짧은 분산 락(여러 인스턴스면). 테스트: 동시 미스 2요청 → DB 1회 호출 단언.
```

# NOVA – Beyond the Mission · حالة المشروع

**آخر تحديث: 2026-11-12** · إصدار الباك-إند `4.0.0` · الفرع `main`

> الملف ده هو المرجع اللي يقول إحنا واقفين فين بالظبط: إيه اللي خلص، إيه اللي
> اتأكدنا منه بالتشغيل مش بالكلام، وإيه اللي فاضل. كل رقم هنا طالع من أمر اتنفذ
> فعلًا على الكود الموجود في المستودع.

---

## 1. الملخص التنفيذي

**الباك-إند خلص.** مفيش مهمة ناقصة، ومفيش endpoint نصّه شغال، ومفيش محاكي
بيرجّع أرقام وهمية من غير ما يقول إنها محاكاة. الخدمة دلوقتي تقدر تشغّل الحملة
كاملة (8 مراحل / 15 مهمة) من أول تجهيز رائد الفضاء لحد تقرير محطة رصد الأرض،
وكل ده من الـ API العام فقط — يعني أي عميل 3D يقدر يلعب اللعبة من غير ما يحتاج
منطق لعب في المتصفح.

| المقياس | القيمة | إزاي اتأكدنا |
| --- | --- | --- |
| المراحل / المهمات | **8 / 15** | `content/missions/*.json` + `scripts/validate_content.py` |
| الإجراءات المتحقّق منها | **95** (93 هدف + 2 إجراء جانبي) | `validate_content.py --strict` |
| عمليات HTTP منشورة | **67** على 64 مسار + WebSocket واحد | `scripts/audit_routes.py --check` |
| مخططات الاستجابة (schemas) | **136** | `docs/openapi.json` (OpenAPI 3.1) |
| أكواد الأخطاء الموثّقة | **25** في `x-error-codes` | `docs/openapi.json` |
| عمليات كتابة محمية | **15** (`require_game_write`) | route audit |
| الاختبارات | **437 اختبار – كلها ناجحة** | `pytest` (بدون شبكة) |
| تغطية الكود | **93%** (6075 جملة، 410 غير مغطاة) | `pytest --cov=app` |
| لعب الحملة كاملة | **15/15 مهمة · 93 إجراء · 0 أخطاء** | `scripts/playthrough.py --in-process` |
| محاكيات على السيرفر | **5** | `app/simulations/` |
| عناصر / موارد / إنجازات | **26 / 4 / 20** (4 مستويات) | `content/items.json`, `resources.json`, `achievements.json` |
| حجم الكود | ~13.1 ألف سطر تطبيق + ~5.7 ألف سطر اختبارات + ~1.1 ألف سطر سكربتات | `wc -l` |
| لغات المحتوى | عربي + إنجليزي في كل مهمة | `title/summary/hint/messages` |

**اللي فاضل في المشروع كله = الفرونت-إند (العميل ثلاثي الأبعاد).** المرحلة دي
مش جزء من التسليم الحالي؛ خطة البدء بها في القسم 12.

---

## 2. اتأكدنا إزاي النهاردة (2026-11-12)

بيئة التحقق: Python 3.11.2 · FastAPI 0.162.5 · Pydantic 2.14.3 · SQLAlchemy 2.0.57
· SQLite (aiosqlite). التثبيت من `requirements-dev.txt` مباشرة.

### 2.1 بوابات الجودة (نفس اللي CI بيعمله)

| البوابة | الأمر | النتيجة |
| --- | --- | --- |
| Lint | `ruff check .` | ✅ All checks passed |
| Format | `ruff format --check .` | ✅ 108 files already formatted |
| Types | `mypy app scripts` | ✅ no issues in 80 source files |
| محتوى المهمات | `validate_content.py --strict` | ✅ 15 missions / 95 actions – 0 errors, 0 warnings |
| مراجعة المسارات | `audit_routes.py --check` | ✅ 0 errors, 0 warnings (24 ملاحظة info) |
| عقد الـ API | `export_openapi.py --check` | ✅ `docs/openapi.json` up to date (67 operations) |
| الاختبارات | `pytest --cov=app` | ✅ 437 passed · TOTAL 93% |
| الحملة | `playthrough.py --in-process --seed 7` | ✅ campaign completed (exit 0) |

ملاحظة تشغيلية: `make check` هو المدخل الرسمي، لكن أداة `make` مش متوفرة في
بيئة التحقق الحالية، فنفّذنا نفس الأوامر حرفيًا واحد واحد (النتائج فوق).
السجل الكامل بالتواريخ وإصدارات المكتبات محفوظ في
[`NOVA/backend/docs/verification-2026-11-12.md`](NOVA/backend/docs/verification-2026-11-12.md).

### 2.2 نتيجة الحملة (كل مهمة اتلعبت عبر HTTP العام)

```
MISSION                        SCORE  GRADE  MISTAKES  STEPS
training_prep                 100.0%      S         0      7
analog_systems_check          100.0%      S         0      4
analog_device_fault           100.0%      S         0      6
analog_emergency              100.0%      S         0      8
analog_power_link             100.0%      S         0      6
analog_comms_blackout         100.0%      S         0      5
analog_water_leak             100.0%      S         0      4
analog_water_recycling        100.0%      S         0      6
mars_arrival                  100.0%      S         0      8
mars_rover_traverse           100.0%      S         0      6
crew_health                   100.0%      S         0      9
freefall_fire                 100.0%      S         0      8
martian_guide                 100.0%      S         0      7
lunar_landing                 100.0%      S         0      7
earth_trends                  100.0%      S         0      6

Missions completed : 15 / 15     Readiness : 100.0 (S)
Actions            : 93 ok / 0 mistakes
```

السكربت بيستخدم **المسارات العامة فقط** — فأي مهمة مش قابلة للإنهاء كانت هتفشل
هنا. ده أقوى دليل إن المحتوى + المحرّك متوافقين.

### 2.3 قاعدة «حركة → تفاعل → نتيجة» اتأكدنا منها على سيرفر شغّال

شغّلنا `uvicorn app.main:app --host 0.0.0.0 --port 8000` وجرّبنا بإيدنا:

```
POST /api/v1/games                                  → game_id + session_token
POST …/missions/training_prep/actions ×7            → 200 لكل خطوة (grade S)
POST …/missions/analog_water_leak/actions
     {"action":"repair_pipe"}  قبل قفل الصمام        → 409 precondition_failed
     "The line is still pressurised – water sprays across the bay.
      Close the valve first."  + details.unmet
GET  …/resources                                    → water 88 (مفيش مكافأة مجانية)
GET  /                                              → 200 (Mission Control, 46.9 KB)
GET  /api/v1/ready                     → status ready · database true · content true
                                         missions_loaded 15 · items_loaded 26
WS   …/games/{id}/stream?after_seq=0   → stream.ready ثم events:
                                         game.created · mission.started · action.succeeded
```

يعني الفشل بيحصل فعلًا، والرسالة بتقول للاعب يعمل إيه، والموارد بتتغير كنتيجة
للإجراء مش كرقم العميل بيبعته.

### 2.4 بيانات NASA الحقيقية (مش fallback)

| الطلب | النتيجة | المصدر |
| --- | --- | --- |
| `POST /api/v1/earth/compare` (nile_delta · lst · 2001-2005 → 2021-2025 · `live=true`) | **`provenance: "live"`** | NASA POWER `T2M` (MODIS MOD11A2 / MERRA-2) |
| نفس الطلب مرة تانية | `provenance: "cache"` | كاش الخدمة (TTL 900s) |
| `GET /api/v1/nasa/apod` | **`provenance: "live"`** | NASA APOD |

النتيجة الحقيقية اللي رجعت: حرارة سطح الأرض في دلتا النيل **+0.89 °C (+4.06%)**
بين الفترتين، التصنيف `degrading`، مع 5 نقاط ساخنة بإحداثياتها، وسرد بالعربي
والإنجليزي. الأرقام دي جاية من الخدمة الفعلية مش من الملف المرفق.

لو الشبكة وقعت أو `DEMO_KEY` اتحدد معدله، نفس الـ endpoint بيرجع
`provenance: "bundled"` مع `live_fallback_reason` مكتوب — اللعب ما بيقفش.

---

## 3. حالة كل طبقة

| الطبقة | الحالة | تغطية الاختبار | ملاحظات |
| --- | :-: | --: | --- |
| `api/v1` (10 وحدات) | ✅ مكتمل | 88–100% | كل عملية لها `response_model` |
| `api/legacy.py` | ✅ متوافق مع Prototype-01 | 94% | deprecated بنفس المحرّك ونفس الحماية |
| `core` (أخطاء، أمان، حدود، كاش، أقفال، routing) | ✅ مكتمل | 76–100% | `routing.py` بيتعامل مع FastAPI ≥ 0.160 |
| `missions` (تعريفات، محرّك، سجل، تحقق) | ✅ مكتمل | 77–94% | `validation.py` أغلب أسطُره رسائل فشل |
| `systems` (موارد، مخزون، أكسجين، نقاط، ربط، إنجازات، أحداث) | ✅ مكتمل | 88–100% | |
| `simulations` (5 محاكيات) | ✅ مكتمل | 94–99% | فيزياء على السيرفر |
| `services` (game / simulation / earth / science) | ✅ مكتمل | 88–99% | |
| `db` + `migrations` | ✅ مكتمل | 88–100% | Alembic = `create_all` متطابقين (اختبار) |
| `integrations/nasa_client` | ✅ مكتمل | 100% | إعادة محاولة + كاش + fallback |
| `schemas` | ✅ مكتمل | 98–100% | 136 مخطط |
| كونسول Mission Control | ✅ مكتمل | 19 اختبار | HTML واحد بدون مكتبات خارجية |
| CI/CD + Docker + Alembic | ✅ مكتمل | 4 اختبارات على الـ workflow | 3 jobs: quality · tests · campaign |
| التوثيق | ✅ مكتمل | بوابة جديدة (القسم 9) | README جذر + دليل الباك-إند + routes + openapi |
| **الفرونت-إند 3D** | ⏳ **مش بدأ** | — | المرحلة الجاية |

---

## 4. المهمات الخمستاشر — كلها متحقّق منها

| # | المرحلة | المهمة | البيئة | أهداف | نتيجة اللعب |
| --: | --- | --- | --- | --: | :-: |
| 1 | Training Facility | تجهيز المهمة | `training_facility` | 7 | S |
| 2 | Earth Analog | فحص أنظمة القاعدة | `earth_analog` | 4 | S |
| 2 | Earth Analog | إصلاح تسريب المياه | `earth_analog` | 4 | S |
| 2 | Earth Analog | توصيل وحدة الطاقة | `earth_analog` | 6 | S |
| 2 | Earth Analog | تشغيل إعادة تدوير المياه | `earth_analog` | 6 | S |
| 2 | Earth Analog | انقطاع الاتصالات | `earth_analog` | 5 | S |
| 2 | Earth Analog | عطل في جهاز علمي | `earth_analog` | 6 | S |
| 2 | Earth Analog | تدريب طوارئ | `earth_analog` | 8 | S |
| 3 | Mars | الوصول للمريخ والاستكشاف | `mars_surface` | 8 | S |
| 3 | Mars | رحلة العربة الطويلة | `mars_surface` | 5 | S |
| 4 | Crew Health | متابعة صحة الطاقم | `medical_bay` | 8 | S |
| 5 | Freefall Fire | حريق في انعدام الجاذبية | `orbital_station` | 7 | S |
| 6 | Martian Guide | أخذ عينات جيولوجية | `mars_lab` | 7 | S |
| 7 | Lunar Landing | محاكي الهبوط على القمر | `simulation_center` | 6 | S |
| 8 | Earth Trends | محطة رصد الأرض | `earth_observation` | 6 | S |

كل مهمة في الملف بتاعها بتحدد: `world_objects` (الأجهزة اللي اللاعب هيمشي لها)،
`animation` و`duration_ms` لكل هدف (اللي العميل هيشغّله)، `requires` (أدوات /
حالة عالم / أهداف سابقة)، `effects` (تغيير العالم والموارد واستهلاك الأدوات)،
`score`، ورسائل النجاح والفشل بالعربي والإنجليزي. `requires_missions` بيبني
شجرة الاعتماديات والمحرّك بيرفض يبدأ مهمة قبل شروطها.

**مثال التسريب (اللي اتأكدنا منه فوق):** `scan_leak` → `close_valve` →
`repair_pipe` → `verify_pressure`. المياه: 88 → 88 → 100 → 100 لتر. لو صلّحت
قبل الصمام: 409 + خسارة عدة إصلاح + خسارة مياه.

---

## 5. الأنظمة اللي بتربط اللعب ببعضه

* **الموارد:** مياه / أكسجين / طاقة / وقود، لكل واحد سقف ومستوى حرج، وبتتصرف
  كنتيجة للإجراءات والتيك (`tick`) — مش بأرقام من العميل. `dt_seconds` بيتقصّ
  على السيرفر فمفيش غش بتسريع الساعة.
* **المخزون واستهلاك الأدوات:** 26 عنصر، الأدوات بتتآكل بالاستخدام، والصيانة
  بتستهلك عدة صيانة أو طاقة القاعدة. حالة الأداة بتأثر على دقة التحليل العلمي.
* **الأكسجين وEVA:** معدل استهلاك مختلف جوّه القاعدة وبرّاها، والبدلة لها
  سلامة، والخروج من مكان مضغوط بيغيّر الحساب.
* **النقاط والتقدير:** كل مهمة لها درجة (S/A/B/C…) وأخطاء ووقت، والتقرير بيفصّل
  الخطوات والنتائج.
* **Mission Continuity:** جودة التجهيز + صيانة الأدوات + الانضباط في الموارد +
  درجة التدريب → درجة جاهزية بتعدّل بداية المراحل الجاية. كل مُعدِّل معاه
  اسم ومصدر ودلتا، فمفيش «مضاعف سحري» مخبي.
* **الأحداث والبث:** `WS /api/v1/games/{id}/stream?after_seq=N` هو **read model**
  فوق سجل الأحداث المحفوظ: انقطاع الاتصال أو مستهلك بطيء (`stream.resync`)
  بيكمّل من المؤشر بتاعه ومفيش حدث بيضيع.
* **الإنجازات:** 20 إنجاز في 4 مستويات مع تقدّم حي للمقفول منها.
* **الحفظ والنقل:** تصدير/استيراد حزمة الحفظ (حالة + تقارير + عينات + تشغيلات)،
  استئناف اللعبة بإعادة إصدار التوكن، حد أقصى لعدد الحملات المحفوظة.

---

## 6. المحاكيات (فيزياء على السيرفر)

| المحاكي | بيحسب إيه | التبسيط المعلن |
| --- | --- | --- |
| `lunar_lander` | جاذبية قمرية 1.62 م/ث²، دفع 0–100%، زاوية اتجاه، وقود، خطوات تكامل ثابتة 0.1 ث، تقييم نعومة الهبوط داخل المنصة | نموذج مبسّط (مستوي رأسي/أفقي)، مش ديناميكا طيران حقيقية |
| `microgravity_fire` | لهب كروي بدون حمل حراري، انتشار محدود بالأكسجين، تأثير التهوية، إطفاء CO₂ وسحب الأكسجين من المقصورة بعده | نموذج تعليمي معاير (مستوحى من Saffire/FLEX)، مش CFD |
| `crew_vitals` | مؤشرات حيوية افتراضية + استجابة لإجراء دعم بالترتيب الصح | **أرقام متخيّلة ومحاكاة تعليمية، مش إرشاد طبي** (التنبيه جوّه كل استجابة) |
| `rover_traverse` | حركة، بطارية، عدّاد مسافة، مخاطر تضاريس، قيادة يدوية أو autopilot للـ waypoints | سرعات معايرة للّعب (العربات الحقيقية بتزحف سنتيمترات/ثانية) |
| `earth_trends` | مقارنة فترتين على شبكة خلايا: إحصاءات، دلتا لكل خلية، نقاط ساخنة، سرد بلغتين | POWER حقيقي للحرارة/المطر، والباقي نموذج تعليمي مُعلن |

كل محاكي بيشتغل بخطوات (`start` → `tick` → `reset`)، بيسحب طاقة من القاعدة،
وبيسجّل كل محاولة في `simulation_runs` عشان التقرير يقارن المحاولات.
محاكي القمر موجود جوّه **Simulation Center** — اللعبة مش بتخلط القمر بالمريخ.

---

## 7. الصدق العلمي (مهم للتسليم الأكاديمي)

كل payload فيه بيانات بيحمل مصدره:

| المصدر | معناه |
| --- | --- |
| `live` | اتجاب من خدمة NASA دلوقتي |
| `cache` | نفس البيانات من كاش الخدمة (TTL واضح) |
| `bundled` | نسخة مرفقة Offline لأن الشبكة/المفتاح مش متاح |
| `simulated` | نموذج تعليمي مبسّط، مش قياس حقيقي |

* طبقات الحرارة (`lst`) والمطر (`precipitation`) بتدعم POWER حقيقي بدون مفتاح.
* طبقات `ndvi` / `aerosol` / `night_lights` نموذج تعليمي **مكتوب عليه كده جوّه
  الـ payload** مش في التوثيق بس.
* الكيمياء المعدنية في `content/science/mars_minerals.json` (6 معادن + 4 مواقع)
 نطاقاتها مبسّطة مستوحاة من نتائج NASA المنشورة، والأطياف مولّدة محاكاة.
* الإجراء الطبي والمؤشرات الحيوية وحريق انعدام الجاذبية كلهم محاكاة تعليمية.
* تأثيرات الموارد بين المراحل هي **توازن لعب** مش فيزياء، وده معلن.

---

## 8. التشغيل والأمان

* **مصادقة:** كل كتابة على لعبة محمية بـ `require_game_write` (15 عملية).
  `NOVA_AUTH_ENABLED=true` ⇒ لازم `X-Session-Token`. مسارات Prototype-01
  القديمة بتشتغل بنفس الحماية (المراجعة كشفت ثغرة تجاوز حقيقية واتقفلت).
* **حدود الطلبات:** rate limit (1800/60s افتراضيًا، الصحة معفاة) + حد لحجم
  الجسم (256 KB) + رفض الحملات فوق الحد الأقصى.
* **Idempotency:** إعادة إرسال نفس الإجراء بنفس المفتاح بترجّع أول نتيجة
  بدل ما تنفّذ مرتين.
* **أخطاء موحّدة:** مِغلاف واحد `{error:{code,message,details},request_id}`
  و25 كود خطأ منشورين في `x-error-codes`.
* **قاعدة البيانات:** SQLite افتراضيًا، أي URL غير متزامن شغال، وفي الإنتاج
  `NOVA_AUTO_CREATE_TABLES=false` + `alembic upgrade head`.
* **النشر:** Dockerfile بيشغّل الترحيلات قبل uvicorn بمستخدم غير root وhealthcheck،
  و`docker-compose.yml` بفوليوم دائم.
* **CORS:** مضبوط مسبقًا لمنافذ Vite ونطاقات المعاينة (`*.e2b.app` وغيرها).

---

## 9. اللي اتعمل في الجلسة دي (استكمال آخر خطوة)

1. **`README.md` في جذر المستودع** — نظرة المشروع كاملة: الفكرة، حالة كل طبقة،
   التشغيل في 3 أوامر، خريطة الملفات، جدول الحملة، ربط أزرار التحكم بالـ
   endpoints، والصدق العلمي. (كان ناقص: الجذر مفيهوش أي README.)
2. **`PROJECT_STATUS.md`** — الملف ده: تقرير حالة مبني على تشغيل فعلي.
3. **بوابة توثيق جديدة** `tests/test_project_docs.py` — بتفشل لو أرقام التوثيق
   في الجذر بقت مختلفة عن حزمة المحتوى أو عقد الـ API المنشور (نفس فلسفة
   بوابة `openapi --check`: التوثيق جزء من التسليم مش ديكور).
4. **تنضيف** `reference/scratch/`: مسحنا نسختي `payloads*.json` (586 KB مكرّرة)
   وسيبنا `capture.py` + README صغير بيشرح إزاي تولّدهم تاني.
5. **إعادة تشغيل كل البوابات** على البيئة الحالية وتسجيل النتيجة (القسم 2).

---

## 10. حدود معروفة وقرارات مؤجلة بوعي

| البند | الوضع | القرار |
| --- | --- | --- |
| 24 عملية لسه فيها حقول `object` حرّة | ملاحظة `info` من المراجعة | مقصودة: تيليمترية متغيّرة + مسارات legacy. العميل بيولّد أنواعها من `docs/openapi.json` |
| ناقل الأحداث و rate-limit محليين للعملية | مش Redis | كفاية لعامل (worker) واحد؛ للتوسّع الأفقي محتاج Redis (موثّق) |
| SQLite افتراضي | مناسب للعبة/العرض | Postgres بتغيير `NOVA_DATABASE_URL` بس |
| `DEMO_KEY` لـ NASA | معدل محدود جدًا | حط `NOVA_NASA_API_KEY` مجاني؛ الـ fallback بيحمي اللعب |
| NDVI / aerosol / night-lights | نموذج تعليمي | بيانات MODIS الحقيقية محتاجة Earthdata login + معالجة tiles |
| حسابات لاعبين / leaderboard عام | مفيش تسجيل دخول | الـ `game_id` نفسه capability سرّي، والتوكن لكل حملة |
| فيزياء المحاكيات | مبسّطة تعليميًا | معلنة في كل payload |

---

## 11. إزاي تشغّل وتتأكد بنفسك

```bash
cd NOVA/backend
python3 -m venv .venv && .venv/bin/pip install -r requirements-dev.txt

# سيرفر + كونسول
.venv/bin/uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
#   /            Mission Control (كونسول بيلعب الـ API)
#   /docs        Swagger UI
#   /api/v1/ready تشخيص الخدمة

# كل البوابات
make check
# أو يدويًا لو make مش متوفر:
.venv/bin/ruff check . && .venv/bin/ruff format --check .
.venv/bin/mypy app scripts
.venv/bin/python scripts/validate_content.py --strict
.venv/bin/python scripts/audit_routes.py --check
.venv/bin/python scripts/export_openapi.py --check
.venv/bin/pytest --cov=app --cov-report=term-missing
.venv/bin/python scripts/playthrough.py --in-process --json campaign.json
```

---

## 12. الخطوة الجاية: الفرونت-إند (مش جزء من التسليم الحالي)

الباك-إند جاهز يستقبل عميل 3D فورًا. الترتيب المقترح:

1. **توليد العميل المطبوع:**
   `npx openapi-typescript NOVA/backend/docs/openapi.json -o src/api/nova.d.ts`
2. **هيكل العميل:** `game/player` (Astronaut, Movement, CameraController)،
   `game/environments` (Mars, EarthAnalog, Moon…)، `game/systems`
   (Interaction, Inventory, Oxygen, Physics)، `game/missions`، `game/ui`.
3. **حلقة اللعب:** `tick` مرة في الثانية تقريبًا (مش كل فريم)، و`E` ينادي
   `…/actions` بـ `Idempotency-Key`، والأنيميشن من `animation` + `duration_ms`
   اللي جايين في تعريف الهدف.
4. **الـ HUD:** من snapshot واحد (`GET /games/{id}`) + بث الأحداث، والأزرار
   تتعطّل من `interactions.unmet` بدل ما اللاعب يفشل عمياني.
5. **الأصول:** Blender → glTF، وRapier للتصادم على العميل فقط (النتائج
   الحاسمة دايمًا من السيرفر).

---

## 13. ملفات التسليم

| الملف | إيه هو |
| --- | --- |
| `repo/` | المستودع كامل بنفس تخطيط GitHub (الكوميت `281b6be` فوق `23692ef v10`) |
| `nova-backend-finish.patch` | باتش التغييرات الجديدة فقط – يتطبّق بـ `git am` |
| `NOVA-backend-final.zip` | نسخة كاملة من شجرة المستودع بعد التعديل (بدون `.venv` ولا كاش) |
| `verification/gates-final.log` | سجل التشغيل الكامل لكل البوابات (خارج المستودع) |
| `verification/campaign.json` | تقرير الحملة بصيغة JSON اللي طلعه الـ playthrough |
| `verification/post-commit.log` | إعادة تشغيل البوابات بعد الكوميت |

```bash
git am nova-backend-finish.patch     # من جذر المستودع
git push origin main                 # محتاج صلاحياتك على GitHub
```

**الخلاصة: الباك-إند مقفول ومتحقّق منه — 15/15 مهمة، 437 اختبار، 93% تغطية،
بيانات NASA حقيقية بتشتغل، والحملة بتتلعب كاملة من الـ API العام.**

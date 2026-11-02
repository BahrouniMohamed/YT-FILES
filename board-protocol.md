# بروتوكول اللوحة

اللوحة = Artifact حيّة بقاعدة بيانات مشتركة فيها أربع مجموعات: staff وtasks وevents وneeds.
التخزين بتوقيت ISO، والعرض للمالك بنظام 12 ساعة.

## المجموعات
**staff** (صف لكل موظف)
id (رمز القسم)، name، level، xp، status (يعمل / متاح / متوقف)، current_task، last_report، updated_at

**tasks**
id (T-0001)، dept، title، status (backlog / doing / review / done / blocked)، size (صغير / متوسط / كبير)، priority (1-3)، due، needs_owner (نعم/لا)، files[]، created_at، updated_at

**events** (سجل يُضاف إليه فقط، ما يُحذف ولا يُعدَّل)
id، at، dept، type (done / blocked / need / xp / rule / note)، task، summary، files[]

**needs** (هذي اللي تظهر في خانة «ينتظرك»)
id، dept، task، kind (موافقة / قرار / معلومة / وصول)، ask (سطر واحد واضح مع الخيارات)، status (open / answered)، answer، asked_at، answered_at

**خانة «ينتظرك»** = كل need حالته open + كل task حالته review وneeds_owner = نعم. مرتبة بالأولوية ثم بالأقدم.

## متى يحدّث القسم
| الحدث | التحديث |
|---|---|
| بداية الجلسة | staff.status = يعمل، وcurrent_task |
| استلام مهمة | task.status = doing |
| ظهور حاجة من المالك | need جديد فورًا، مو آخر الجلسة، وevent من نوع need |
| تسليم جاهز ومطابق لـ«خلص» | task.status = review، وevent من نوع done فيه الملفات |
| توقف | task.status = blocked، وevent من نوع blocked، وneed إذا المالك يقدر يفكّه |
| نهاية الجلسة | staff.status = متاح، وlast_report = مسار التقرير |

## كيف يحدّث
1. **مباشر**: إذا الجلسة فيها أداة Artifact مع صلاحية الكتابة على قاعدة اللوحة، يكتب القسم مباشرة.
2. **بديل (outbox)**: إذا ما قدر، يضيف أسطر JSON في team/board/outbox/<الرمز>.jsonl، سطر لكل تحديث:
```
{"collection":"tasks","op":"update","id":"T-0001","data":{"status":"review"}}
{"collection":"needs","op":"add","data":{"kind":"قرار","ask":"أي مجال من الثلاثة؟ أ / ب / ج","task":"T-0001"}}
```
المدير التنفيذي يدفعها للوحة في ختام اليوم (أو قبل ملخص الصباح) وينقل الملف إلى board/outbox/processed/.

## من يكتب إيش
- كل قسم: صف staff الخاص به (ما عدا level وxp)، ومهامه، وأحداثه، وحاجاته.
- المدير التنفيذي فقط: level وxp وأحداث xp وrule، وإغلاق needs بعد جواب المالك، ودفع outbox.

## ممنوع
- كلمات سر أو مفاتيح أو رموز دخول في أي حقل.
- حذف أو تعديل events.
- need بصيغة غامضة. كل need سطر واحد مفهوم من غير ما المالك يفتح ملف.

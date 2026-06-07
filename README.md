# 🔥 RVAPatcher - Runtime ARM64 Memory Patcher for Android

## 📖什么是 RVAPatcher؟

RVAPatcher هي مكتبة Android احترافية لتعديل الذاكرة في وقت التشغيل (Runtime Patching) لمعمارية ARM64. تم تصميمها خصيصاً لتعديل تعليمات الدوال في مكتبة `libil2cpp.so` **دون تعديل الملف الأصلي على القرص**، مما يجعلها أداة قوية وخفية في نفس الوقت.

---

## ✨ الميزات الرئيسية (Features)

| الميزة | الوصف |
|--------|-------|
| 🔄 **Runtime Patching** | تعديل الذاكرة أثناء تشغيل التطبيق، بدون الحاجة لإعادة التشغيل |
| 🎯 **Custom Instructions per RVA** | كل عنوان (RVA) له تعليماته المخصصة والمستقلة |
| ⏹️ **Toggle ON/OFF** | تشغيل وإيقاف التعديل فوراً (مع مكتبة AdemX_Final) |
| 🕶️ **No Traces / Stealth** | يمسح آثاره من الذاكرة تلقائياً بعد الانتهاء |
| 📁 **External Configuration** | ملف `MOD.SH` خارجي لتعديل RVAs بدون إعادة تجميع |
| 🔀 **Multiple Instructions** | يدعم تعليمات متعددة تصل إلى 64 بايت لكل RVA |
| ⏱️ **Delay Support** | تأخير زمني قبل التعديل (مثل `/5` لانتظار 5 ثوانٍ) |
| 📝 **Inline Comments** | دعم التعليقات في ملف التكوين (أسطر تبدأ بـ `#`) |
| 🧹 **Memory Cleanup** | تنظيف كامل للذاكرة من البيانات الحساسة بعد التنفيذ |
| 🎲 **Obfuscation** | تشويش بسيط لتعقيد التحليل |

---

## 🏗️ المواصفات التقنية (Technical Specifications)

| المواصفة | القيمة |
|----------|--------|
| **اللغة** | C/C++ (Android NDK) |
| **المعمارية** | ARM64 (AArch64) فقط |
| **الحد الأقصى للتعليمات** | 64 بايت لكل RVA (16 تعليمة ARM64) |
| **الحد الأقصى للـ RVAs** | 100 عنوان كحد أقصى |
| **ملف التكوين** | `MOD.SH` (يُوضع في مجلد files التطبيق) |
| **التأخير الزمني** | من 1 إلى 3600 ثانية |
| **الذاكرة المستخدمة** | أقل من 1MB |
| **صلاحيات مطلوبة** | لا يحتاج صلاحيات خاصة (يعمل ضمن التطبيق) |
| **Root مطلوب؟** | لا |
| **Android SDK** | 21+ (Android 5.0) |

---

## 🧠 طريقة العمل (How It Works)

1. المكتبة تُحقن في التطبيق
2. JNI_OnLoad → تشغيل Thread خلفي
3. انتظار تحميل libil2cpp.so في الذاكرة (فحص /proc/self/maps)
4. قراءة ملف MOD.SH من مجلد files التطبيق
5. تحليل التعليمات المخصصة لكل RVA
6. تطبيق التعديلات على الذاكرة عبر mprotect + clear_cache
7. تنظيف الآثار ومسح البيانات الحساسة من الذاكرة

```

---

## 📂 تنسيق ملف MOD.SH (Config File Format)

ملف `MOD.SH` يجب أن يكون في المسار:
```

/storage/emulated/0/Android/data/<package_name>/files/MOD.SH

```

### الصيغة العامة:

```bash
# هذا تعليق - المكتبة تتجاهله

/10                         # تأخير 10 ثوانٍ قبل التعديل

0x874CD0: 20 00 80 52 C0 03 5F D6    # RVA + تعليمات مخصصة

0x874CD4: C0 03 5F D6                # RVA مع RET فقط

0x874CD8                              # RVA بدون تعليمات (يستخدم RET الافتراضي)

# التنسيق القديم مدعوم أيضاً للتوافق
0x874CDC
0x874CE0
```

القواعد:

القاعدة الشرح
# بداية تعليق، المكتبة تتجاهل السطر بالكامل
/رقم تأخير زمني بالثواني (مثال /5 = انتظر 5 ثوانٍ)
0xRVA RVA فقط (يستخدم التعليمات الافتراضية C0 03 5F D6)
0xRVA: BYTES RVA مع تعليمات مخصصة (بايتات hex مفصولة بمسافات)
سطر فارغ يتم تجاهله

---

🔧 التعليمات المدعومة (Supported ARM64 Instructions)

الغرض البايتات (Hex) عدد البايتات الشرح
RET C0 03 5F D6 4 تعطيل الدالة بالكامل (ترجع فوراً)
TRUE (1) 20 00 80 52 C0 03 5F D6 8 إرجاع القيمة 1 (true)
FALSE (0) 00 00 80 52 C0 03 5F D6 8 إرجاع القيمة 0 (false)
NOP 1F 20 03 D5 4 لا تفعل شيئاً (تخطي)
MOV W0, #X XX 00 80 52 4 وضع قيمة X في سجل الإرجاع (مع RET)
قيمة 100 64 00 80 52 C0 03 5F D6 8 إرجاع 100
قيمة 999 E7 03 80 52 C0 03 5F D6 8 إرجاع 999
قيمة 9999 0F 27 80 52 C0 03 5F D6 8 إرجاع 9999

تعليمات إضافية متقدمة (32-64 بايت):

يمكنك كتابة أي تسلسل تعليمات ARM64 صحيح يصل إلى 64 بايت، مثل:

```bash
# حفظ الإطار ثم استدعاء دالة ثم العودة
0x12345678: FD 7B BF A9 FD 03 00 91 00 00 80 52 C0 03 5F D6 FD 7B C5 A8 C0 03 5F D6
```

---

🚀 طريقة الاستخدام (Usage Guide)

الخطوة 1: إضافة المكتبة للمشروع

ضع ملف librvapatcher.so في مجلد المشروع المناسب:

```gradle
// build.gradle (وحدة التطبيق)
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}

dependencies {
    // لا توجد تبعيات خاصة
}
```

الخطوة 2: استدعاء المكتبة في الكود

```java
// في أي مكان في التطبيق (مثل MainActivity.onCreate)
static {
    System.loadLibrary("rvapatcher");
    // المكتبة تعمل تلقائياً! لا حاجة لاستدعاء دوال إضافية
}
```

الخطوة 3: إنشاء ملف MOD.SH

أنشئ ملفاً نصياً باسم MOD.SH بالمحتوى التالي:

```bash
# MOD.SH - تعديل دوال الحماية في Free Fire مثال

/5

# تعطيل دوال فحص التوقيع
0x1432554: C0 03 5F D6
0x14343f8: C0 03 5F D6

# تعديل دوال العملات (إرجاع قيمة كبيرة)
0x1560000: 20 00 80 52 C0 03 5F D6
0x1561000: 64 00 80 52 C0 03 5F D6

# NOP للتجربة
0x1456780: 1F 20 03 D5
```

الخطوة 4: وضع الملف في المسار الصحيح

ضع ملف MOD.SH في:

```
/storage/emulated/0/Android/data/com.your.game/files/MOD.SH
```

الخطوة 5: تشغيل التطبيق

شغّل التطبيق كالمعتاد. المكتبة ستعمل تلقائياً في الخلفية.

---

📋 متطلبات التشغيل (Requirements)

المتطلب التفاصيل
نظام التشغيل Android 5.0 (API 21) أو أحدث
المعمارية ARM64 فقط (لا يدعم x86 أو ARM32)
المكتبة المستهدفة libil2cpp.so (يمكن تعديل الكود لاستهداف مكتبات أخرى)
الصلاحيات لا يحتاج صلاحيات خاصة
Root غير مطلوب

---

🛡️ ميزات الحماية والتمويه (Anti-Detection Features)

الميزة الشرح
No APK Modification لا يغير ملف APK الأصلي، التوقيع يبقى سليماً
Memory Traces Cleanup يمسح التعليمات والبيانات الحساسة من الرام بعد التنفيذ
Random Obfuscation تشويش عشوائي لتعقيد التحليل
No Logs نسخة Silent Mode لا تكتب أي تقارير أو ملفات
Volatile Pointers استخدام متغيرات عابرة لمنع تحسينات المحول البرمجي

---

📁 بنية الكود الأساسية (Core Code Structure)

```cpp
// main.c
#include <jni.h>
#include <pthread.h>
#include <sys/mman.h>

// ثوابت التكوين
#define MAX_RVAS 100
#define MAX_INST 64
#define MOD_FILE "MOD.SH"

// هيكل التعليمات المخصصة لكل RVA
typedef struct {
    uintptr_t rva;
    unsigned char instruction[MAX_INST];
    int inst_len;
    int has_custom;
} PatchEntry;

// الوظائف الرئيسية
void detect_package();           // كشف حزمة التطبيق
int load_rvas();                 // قراءة ملف MOD.SH
int patch_address();             // تعديل عنوان معين
void patch_all();                // تعديل جميع RVAs
void clean_memory_traces();      // تنظيف الآثار
```

---

⚠️ التنبيهات والتحذيرات (Warnings & Disclaimer)

📢 تنبيه مهم

هذه المكتبة للأغراض التعليمية والبحثية فقط.

· ❌ لا تستخدمها في اختراق تطبيقات الآخرين
· ❌ لا تستخدمها في الغش في الألعاب عبر الإنترنت
· ❌ لا تستخدمها في تعطيل الإعلانات أو تجاوز التراخيص
· ✅ استخدمها فقط على تطبيقاتك الخاصة أو في بيئات اختبارية

أنت المسؤول الوحيد عن أي استخدام غير قانوني أو غير أخلاقي لهذه الأداة.

MIT License

Copyright (c) 2025 ADEM

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

👨‍💻 المطور (Author)

ADEM - باحث ومطور أمن سيبراني

· متخصص في ARM64 Reverse Engineering
· مطور أدوات Android Runtime Patching
· مهتم بأمن التطبيقات والهندسة العكسية

---

⭐ Support the Project

إذا أعجبتك المكتبة، لا تنسى وضع ⭐ على GitHub!

---

📅 آخر تحديث: 2025

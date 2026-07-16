# CVE-2026-35204 - Helm Plugin Path Traversal Arbitrary File Write

## Title
CVE-2026-35204 - Helm Plugin Path Traversal Arbitrary File Write

## Executive Summary
ثغرة أمنية في **Helm** (مدير حزم Kubernetes) تسمح بكتابة ملفات في مواقع عشوائية بنظام الملفات عند تثبيت أو تحديث إضافة Helm (Helm Plugin) مصممة خصيصاً. تنشأ الثغرة من عدم التحقق الكافي من المسارات في ملف `plugin.yaml` للإضافة.

يمكن لمهاجم إنشاء إضافة Helm ضارة تحتوي على ملف `plugin.yaml` بمسار متجاوز (`../`) في حقل المسار، مما يؤدي إلى كتابة ملفات خارج الدليل المخصص للإضافة. هذا يمكن أن يؤدي إلى **تنفيذ أوامر عشوائية** عند كتابة ملفات في مواقع حساسة مثل `~/.bashrc` أو `/etc/systemd/` أو ملفات Helm hooks.

| العنصر | التفاصيل |
|--------|----------|
| **CVE** | CVE-2026-35204 |
| **CVSS v3.1** | 8.6 (High) |
| **المتجه** | CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N |
| **النوع** | Path Traversal (CWE-22) / Arbitrary File Write |
| **المنتج** | Helm (Kubernetes Package Manager) |
| **الإصدارات المتأثرة** | من 4.0.0 إلى 4.1.3 |
| **الإصدار المُصحَّح** | 4.1.4+ |
| **التأثير** | كتابة ملفات عشوائية → تنفيذ أوامر |

| Attribute | Value |
|---|---|
| Identifier | CVE-2026-35204 |
| CVSS / Severity | 8.6 |
| Weakness Class | CWE-22) / Arbitrary File Write |
| Affected Scope | من 4.0.0 إلى 4.1.3 |
| Fixed Version | 4.1.4+ |
## Technical Analysis
The weakness was assessed from an application-security and infrastructure-risk perspective. The core issue is classified as **Path Traversal** and was documented in a sanitized form suitable for public portfolio publication.

Helm هو مدير الحزم الرسمي لـ Kubernetes، يستخدم لتثبيت وإدارة التطبيقات المعبأة كـ Charts. يدعم Helm نظام **الإضافات (Plugins)** لتوسيع وظائفه.

## Impact
- Unauthorized file access or modification outside the intended application boundary.
- Exposure of configuration data, credentials, and potential execution-chain enablement.

## Remediation
| الإجراء | الأولوية | الوصف |
|---------|----------|-------|
| **تحديث Helm** | 🟢 فوري | الترقية إلى Helm 4.1.4 أو أحدث |
| **تدقيق الإضافات** | 🟡 مهم | مراجعة أي إضافات مثبتة حالياً بحثاً عن Path Traversal |
| **عدم تثبيت إضافات غير موثوقة** | 🟡 مهم | استخدام إضافات من مصادر موثوقة فقط |
| **مراجعة plugin.yaml** | 🟡 مهم | فحص ملف plugin.yaml لأي إضافة قبل تثبيتها |
| **الحد من الصلاحيات** | 🔵 مستمر | تشغيل Helm بأقل الصلاحيات الممكنة |

## Lessons Learned & Mitigation Strategy
- Treat every integration boundary as untrusted, especially when application logic forwards user-controlled values to filesystems, shells, parsers, or external tools.
- Security reviews should validate the complete exploit chain, not only the first vulnerable endpoint; low-severity misconfigurations can become critical when chained.
- Public-facing documentation should describe risk, root cause, and remediation without exposing operational identifiers, credentials, or reusable exploitation artifacts.
- Defensive controls should combine preventive validation, runtime least privilege, telemetry, and patch governance to reduce both exploitability and blast radius.

## Publication Sanitization Notes
- Sensitive infrastructure identifiers, IP addresses, hostnames, credentials, hashes, and e-mail addresses were replaced with explicit placeholders.
- Reusable operational evidence was minimized or abstracted to keep the document suitable for public GitHub publication.
- The document uses a consultant-style structure aligned with common web security testing report practices such as OWASP WSTG reporting expectations.

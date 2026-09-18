# Kubernetes Deploy Starter

Everything one web app needs to run on Kubernetes, written out in full and explained.

---

## In English

Most Kubernetes examples show you one file and stop there. Then you try it on a real cluster and find out you also needed health checks, resource limits, a way to reach the app from outside, and a second copy of everything for testing. This repo has all of that, for one small web app.

### What is inside

**The base** (`base/`) is the app described once:

- `deployment.yaml` runs the app, with health checks and memory limits
- `service.yaml` gives it one fixed address inside the cluster
- `ingress.yaml` lets people reach it from outside, over https
- `hpa.yaml` adds more copies when the app gets busy, removes them when it is quiet
- `configmap.yaml` holds settings that are not secret
- `pdb.yaml` stops the cluster from taking down all copies at the same time during maintenance

**The overlays** (`overlays/dev` and `overlays/prod`) are the small differences between testing and production: how many copies, how much memory, and which web address. The app itself is described only once.

### Try it

```bash
# See what would be created, without creating it
kubectl kustomize overlays/dev

# Create it for real
kubectl apply -k overlays/dev

# Watch it come up
kubectl get pods -n demo-dev -w
```

To remove everything again:

```bash
kubectl delete -k overlays/dev
```

### The parts that people usually forget

**Health checks.** There are two, and they are not the same thing. The *readiness* check answers "can this copy take traffic right now". The *liveness* check answers "is this copy stuck and needs a restart". If you only set liveness, Kubernetes sends users to a copy that is still starting. If you set liveness too aggressively, it restarts a healthy app that is only slow.

**Resource requests and limits.** The *request* is what the app is promised. The *limit* is the ceiling. Without a request, Kubernetes does not know where to place the app and may put it on a full node. Without a limit, one leaking app can take the memory of everything else on the node.

**Do not run as root.** The security section in the deployment makes the app run as a normal user with a read only file system. If someone breaks into the app, they land in a box with nothing to take.

**A budget for disruption.** `pdb.yaml` tells the cluster that at least one copy must stay alive. Without it, a node upgrade can stop every copy at the same moment and your site goes down during planned maintenance.

### What you need

- A Kubernetes cluster, version 1.25 or newer
- `kubectl` 1.25 or newer, which already has kustomize built in
- An ingress controller in the cluster, for example nginx, if you want the outside address to work
- The metrics server, if you want the auto scaling to work

### The deploy workflow

`.github/workflows/deploy.yml` checks that the files are valid on every push. The deploy step is written out but switched off, because it needs a cluster and a secret. Read the comments inside before turning it on.

---

## بالعربي

معظم أمثلة Kubernetes تعرض عليك ملفاً واحداً ثم تتوقف. وعندما تجرّبها على كلستر حقيقي تكتشف أنك كنت تحتاج أيضاً فحوصات صحة، وحدود موارد، وطريقة للوصول إلى التطبيق من الخارج، ونسخة ثانية من كل شيء للاختبار. هذا الريبو فيه كل ذلك، لتطبيق ويب صغير واحد.

### ما الموجود بالداخل

**القاعدة** (`base/`) هي وصف التطبيق مرة واحدة:

- `deployment.yaml` يشغّل التطبيق، مع فحوصات صحة وحدود ذاكرة
- `service.yaml` يعطيه عنواناً ثابتاً واحداً داخل الكلستر
- `ingress.yaml` يتيح للناس الوصول إليه من الخارج عبر https
- `hpa.yaml` يضيف نسخاً إضافية عندما ينشغل التطبيق، ويزيلها عندما يهدأ
- `configmap.yaml` يحمل الإعدادات غير السرية
- `pdb.yaml` يمنع الكلستر من إيقاف كل النسخ في نفس الوقت أثناء الصيانة

**الطبقات** (`overlays/dev` و `overlays/prod`) هي الفروق الصغيرة بين الاختبار والإنتاج: كم نسخة، وكم ذاكرة، وأي عنوان ويب. أما التطبيق نفسه فموصوف مرة واحدة فقط.

### جرّبه

```bash
# شاهد ماذا سيُنشأ، بدون إنشائه
kubectl kustomize overlays/dev

# أنشئه فعلياً
kubectl apply -k overlays/dev

# راقبه وهو يعمل
kubectl get pods -n demo-dev -w
```

### الأجزاء التي ينساها الناس عادة

**فحوصات الصحة.** هما اثنان، وليسا شيئاً واحداً. فحص *الجاهزية* يجيب عن سؤال: هل تستطيع هذه النسخة استقبال الزوار الآن؟ وفحص *الحياة* يجيب عن: هل هذه النسخة عالقة وتحتاج إعادة تشغيل؟ إذا ضبطت فحص الحياة فقط، فسيرسل Kubernetes المستخدمين إلى نسخة ما زالت تبدأ.

**الموارد المطلوبة والحدود.** *المطلوب* هو ما وُعِد به التطبيق. و*الحد* هو السقف. بدون تحديد المطلوب، لا يعرف Kubernetes أين يضع التطبيق وقد يضعه على جهاز ممتلئ. وبدون حد، يستطيع تطبيق واحد مسرّب أن يأكل ذاكرة كل ما حوله.

**لا تشغّله كـ root.** قسم الأمان في ملف النشر يجعل التطبيق يعمل كمستخدم عادي وبنظام ملفات للقراءة فقط. فإذا اخترق أحد التطبيق، يجد نفسه في صندوق لا شيء فيه.

**ميزانية للتوقف.** ملف `pdb.yaml` يخبر الكلستر أن نسخة واحدة على الأقل يجب أن تبقى حية. بدونه، قد يوقف تحديث الأجهزة كل النسخ في نفس اللحظة فيتوقف موقعك أثناء صيانة مخطط لها.

### ما الذي تحتاجه

- كلستر Kubernetes إصدار 1.25 أو أحدث
- أداة `kubectl` إصدار 1.25 أو أحدث
- متحكم ingress في الكلستر، مثل nginx، إذا أردت أن يعمل العنوان الخارجي
- خادم المقاييس (metrics server) إذا أردت أن يعمل التوسع التلقائي

## 1) Patch diff (disable the HTTP-blocking logic)

Save this as `disable-http-enforcement.patch` and apply with `git apply disable-http-enforcement.patch`.

```diff name=disable-http-enforcement.patch
diff --git a/java/com/google/copybara/util/RepositoryUtil.java b/java/com/google/copybara/util/RepositoryUtil.java
index cf314f8..ece4dcf 100644
--- a/java/com/google/copybara/util/RepositoryUtil.java
+++ b/java/com/google/copybara/util/RepositoryUtil.java
@@ -27,9 +27,8 @@ public class RepositoryUtil {
   /**
    * Verify that a repo URL is not plain HTTP
    */
   public static String validateNotHttp(String url) throws ValidationException {
-    checkCondition(!url.startsWith("http://"),
-        "URL '%s' is not valid - should be using https.", url);
+    // HTTP validation disabled: allow plain http:// repository URLs.
     return url;
   }
 
   private RepositoryUtil() {
```

Optional cleanup (not required): remove now-unused static import of `checkCondition`.

---

## 2) Fedora setup (JDK + Bazel + build tools)

```bash
# Core build deps
sudo dnf install -y git curl unzip zip gcc gcc-c++ make patch which

# Java 21 (recommended by current Copybara docs)
sudo dnf install -y java-21-openjdk java-21-openjdk-devel

# Verify Java
java -version
javac -version
```

Install Bazelisk (recommended wrapper; auto-manages Bazel version):
```bash
mkdir -p ~/.local/bin
curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 \
  -o ~/.local/bin/bazel
chmod +x ~/.local/bin/bazel
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify
bazel version
```

---

## 3) Clone, patch, build uberjar

```bash
git clone https://github.com/google/copybara.git
cd copybara

# apply your patch
git apply /path/to/disable-http-enforcement.patch

# build deploy jar
bazel build //java/com/google/copybara:copybara_deploy.jar
```

Resulting artifact (typically):
```bash
ls -lh bazel-bin/java/com/google/copybara/copybara_deploy.jar
```

Run:
```bash
java -jar bazel-bin/java/com/google/copybara/copybara_deploy.jar --help
```

---

## 4) If build fails on Fedora

Try forcing Java 21 for Bazel runtime:
```bash
echo 'startup --host_jvm_args=-Djava.net.preferIPv4Stack=true' >> .bazelrc.local
echo 'run --java_runtime_version=remotejdk_21' >> .bazelrc.local
echo 'build --java_runtime_version=remotejdk_21' >> .bazelrc.local
```

Then:
```bash
bazel clean --expunge
bazel build //java/com/google/copybara:copybara_deploy.jar
`

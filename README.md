# Overview

This project demonstrates strange behavior of IDE when running Gradle Daemon with environment
variables.

# Prerequisites

- MacOS Ventura 13.7.8 (22H730)
- Preinstalled Android Studio Quail 1 | 2026.1.1 Patch 1
- Preinstalled python 3.11
```shell
brew install python@3.11
```
- Android Studio must be select on JDK 21 (JAVA_VERSION="21.0.10" JAVA_RUNTIME_VERSION="
  21.0.10+-117844309-b1163.108" IMPLEMENTOR="JetBrains s.r.o.")

# Scenario for reproducing bug

- Open this project in IDE
- Click double command for creating Gradle configuration
- In open window write next command './gradlew :app:assemble --quiet'
- Click Enter or Run command from IDE

> Scenario video located [here](resources/scenario-with-jdk-21.webp)

**Expecting behavior:**
Assemble command successfully complete.

**Actual behavior:**
Assemble command could not find python3.11 bin.

# Strange behavior

If you repeat scenario with only one change, namely changing JDK to 17 bug suddenly disappears.

> Video with fix bug [here](resources/scenario-with-jdk-17.webp)

The conclusion seems obvious: something has changed in Java Development Kit, most likely the
behavior of ProcessBuilder.
What's strange is that I couldn't detect any difference between the GradleDaemon environment
variables when running JDK with versions 21 and 17.
In both cases PATH variable is passed to the process in a truncated form.

<details>
  <summary>Environment variables on JDK 21</summary>

```text
Executing tasks: [:app:assemble] in project /Users/iosephknecht/AndroidStudioProjects/process-path-playground

Absolute python path:
/usr/local/bin/python3.11

Environment variables for Gradle Daemon with pid = 1748
PID   TT  STAT      TIME COMMAND
1748   ??  Rs     1:25.77 /Applications/Android Studio.app/Contents/jbr/Contents/Home/bin/java --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.prefs/java.util.prefs=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.prefs/java.util.prefs=ALL-UNNAMED --add-opens=java.base/java.nio.charset=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.xml/javax.xml.namespace=ALL-UNNAMED -XX:MaxMetaspaceSize=384m -XX:+HeapDumpOnOutOfMemoryError -Xms256m -Xmx512m -Dfile.encoding=UTF-8 -Duser.country=US -Duser.language=en -Duser.variant -cp /Users/iosephknecht/.gradle/wrapper/dists/gradle-8.11.1-bin/bpt9gzteqjrbo1mjrsomdt32c/gradle-8.11.1/lib/gradle-daemon-main-8.11.1.jar -javaagent:/Users/iosephknecht/.gradle/wrapper/dists/gradle-8.11.1-bin/bpt9gzteqjrbo1mjrsomdt32c/gradle-8.11.1/lib/agents/gradle-instrumentation-agent-8.11.1.jar org.gradle.launcher.daemon.bootstrap.GradleDaemon 8.11.1 HOME=/Users/iosephknecht SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.oGhUznR78V/Listeners COMMAND_MODE=unix2003 PATH=/usr/bin:/bin:/usr/sbin:/sbin XPC_SERVICE_NAME=application.com.google.android.studio.12884955856.12884956782 SHELL=/bin/zsh __CFBundleIdentifier=com.google.android.studio LOGNAME=iosephknecht TMPDIR=/var/folders/l7/jzkb9z9n3_x7ndrfmh20zxg80000gn/T/ USER=iosephknecht XPC_FLAGS=0x0 __CF_USER_TEXT_ENCODING=0x1F5:0x0:0x0

Environment variables from System.getenv:
HOMEBREW_PREFIX = /usr/local
COMMAND_MODE = unix2003
LC_CTYPE = en_US.UTF-8
INFOPATH = /usr/local/share/info:
SHELL = /bin/zsh
TMPDIR = /var/folders/l7/jzkb9z9n3_x7ndrfmh20zxg80000gn/T/
__CFBundleIdentifier = com.google.android.studio
HOME = /Users/iosephknecht
SSH_AUTH_SOCK = /private/tmp/com.apple.launchd.oGhUznR78V/Listeners
HOMEBREW_REPOSITORY = /usr/local/Homebrew
OLDPWD = /
XPC_SERVICE_NAME = application.com.google.android.studio.12884955856.12884956782
PATH = /usr/local/bin:/usr/local/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin
LOGNAME = iosephknecht
USER = iosephknecht
__CF_USER_TEXT_ENCODING = 0x1F5:0x0:0x0
XPC_FLAGS = 0x0
HOMEBREW_CELLAR = /usr/local/Cellar
FPATH = /usr/local/share/zsh/site-functions:/usr/local/share/zsh/site-functions:/usr/share/zsh/site-functions:/usr/share/zsh/5.9/functions


FAILURE: Build failed with an exception.

* What went wrong:
  Execution failed for task ':app:somePythonWork'.
> A problem occurred starting process 'command 'python3.11''

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.

BUILD FAILED in 3s
```

</details>

<details>
  <summary>Environment variables on JDK 17</summary>

```text
23:13:39: Executing ':app:assemble --quiet'…

Executing tasks: [:app:assemble] in project /Users/iosephknecht/AndroidStudioProjects/process-path-playground

Absolute python path:
/usr/local/bin/python3.11

Environment variables for Gradle Daemon with pid = 2960
PID   TT  STAT      TIME COMMAND
2960   ??  Ss     2:24.10 /Users/iosephknecht/Library/Java/JavaVirtualMachines/corretto-17.0.19/Contents/Home/bin/java --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.prefs/java.util.prefs=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.prefs/java.util.prefs=ALL-UNNAMED --add-opens=java.base/java.nio.charset=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.xml/javax.xml.namespace=ALL-UNNAMED -XX:MaxMetaspaceSize=384m -XX:+HeapDumpOnOutOfMemoryError -Xms256m -Xmx512m -Dfile.encoding=UTF-8 -Duser.country=US -Duser.language=en -Duser.variant -cp /Users/iosephknecht/.gradle/wrapper/dists/gradle-8.11.1-bin/bpt9gzteqjrbo1mjrsomdt32c/gradle-8.11.1/lib/gradle-daemon-main-8.11.1.jar -javaagent:/Users/iosephknecht/.gradle/wrapper/dists/gradle-8.11.1-bin/bpt9gzteqjrbo1mjrsomdt32c/gradle-8.11.1/lib/agents/gradle-instrumentation-agent-8.11.1.jar org.gradle.launcher.daemon.bootstrap.GradleDaemon 8.11.1 HOME=/Users/iosephknecht SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.oGhUznR78V/Listeners COMMAND_MODE=unix2003 PATH=/usr/bin:/bin:/usr/sbin:/sbin XPC_SERVICE_NAME=application.com.google.android.studio.12884955856.12884956782 SHELL=/bin/zsh __CFBundleIdentifier=com.google.android.studio LOGNAME=iosephknecht TMPDIR=/var/folders/l7/jzkb9z9n3_x7ndrfmh20zxg80000gn/T/ USER=iosephknecht XPC_FLAGS=0x0 __CF_USER_TEXT_ENCODING=0x1F5:0x0:0x0

Environment variables from System.getenv:
__CFBundleIdentifier = com.google.android.studio
PATH = /usr/local/bin:/usr/local/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin
SHELL = /bin/zsh
HOMEBREW_CELLAR = /usr/local/Cellar
OLDPWD = /
USER = iosephknecht
HOMEBREW_PREFIX = /usr/local
TMPDIR = /var/folders/l7/jzkb9z9n3_x7ndrfmh20zxg80000gn/T/
COMMAND_MODE = unix2003
SSH_AUTH_SOCK = /private/tmp/com.apple.launchd.oGhUznR78V/Listeners
XPC_FLAGS = 0x0
FPATH = /usr/local/share/zsh/site-functions:/usr/local/share/zsh/site-functions:/usr/share/zsh/site-functions:/usr/share/zsh/5.9/functions
__CF_USER_TEXT_ENCODING = 0x1F5:0x0:0x0
LOGNAME = iosephknecht
HOMEBREW_REPOSITORY = /usr/local/Homebrew
LC_CTYPE = en_US.UTF-8
XPC_SERVICE_NAME = application.com.google.android.studio.12884955856.12884956782
HOME = /Users/iosephknecht
INFOPATH = /usr/local/share/info:

Python 3.11.15

Build Analyzer results available
23:13:44: Execution finished ':app:assemble --quiet'.
```

</details>

This raises two questions:

- Why do I get the full path when requesting System.getenv, which includes the path to Python?
- Apparently, JDK previously had an undocumented behavior, and IDE relied on it?

# Workaround fix

- Open IDE from terminal
```shell
  open -a /Applications/Android\ Studio.app`
```
- Share `PATH` env variable for GUI app
```shell
sudo launchctl config user path "$PATH"
```




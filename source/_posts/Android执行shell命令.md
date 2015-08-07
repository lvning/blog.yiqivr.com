title: Android执行shell命令
date: 2015-01-01 07:01:15
tags: [Android]
---
希望让Android系统执行shell命令，我们主要使用Runtime.exec()方法，比如我们想查看根目录可以用如下方法：

```
try {
// Executes the command.
Process process = Runtime.getRuntime().exec("/system/bin/ls /sdcard");
// Reads stdout.// NOTE: You can write to stdin of the command using
// process.getOutputStream().
BufferedReader reader = new BufferedReader(
new InputStreamReader(process.getInputStream()));
int read;
char[] buffer = new char[4096];
StringBuffer output = new StringBuffer();
while ((read = reader.read(buffer)) &gt; 0) {
output.append(buffer, 0, read);
}
reader.close();
// Waits for the command to finish.
process.waitFor();
return output.toString();
} catch (IOException e) {
throw new RuntimeException(e);
} catch (InterruptedException e) {
throw new RuntimeException(e);
}
```

或者使用这个方法（此方法在Android2.3版本经测试无效）：

```
try {
// android.os.Exec is not included in android.jar so we need to use reflection.
Class&lt;?&gt; execClass = Class.forName("android.os.Exec");
Method createSubprocess = execClass.getMethod("createSubprocess", String.class, String.class, String.class, int[].class);
Method waitFor = execClass.getMethod("waitFor", int.class);
// Executes the command.
// NOTE: createSubprocess() is asynchronous.
int[] pid = new int[1];
FileDescriptor fd = (FileDescriptor)createSubprocess.invoke(null, "/system/bin/ls", "/sdcard", null, pid);
// Reads stdout.
// NOTE: You can write to stdin of the command using new FileOutputStream(fd).
FileInputStream in = new FileInputStream(fd);
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String output = "";
try {
String line;
while ((line = reader.readLine()) != null) {
output += line + "\n";
}
} catch (IOException e) {
// It seems IOException is thrown when it reaches EOF.
}
// Waits for the command to finish.
waitFor.invoke(null, pid[0]);
return output;
} catch (ClassNotFoundException e) {
throw new RuntimeException(e.getMessage());
} catch (SecurityException e) {
throw new RuntimeException(e.getMessage());
} catch (NoSuchMethodException e) {
throw new RuntimeException(e.getMessage());
} catch (IllegalArgumentException e) {
throw new RuntimeException(e.getMessage());
} catch (IllegalAccessException e) {
throw new RuntimeException(e.getMessage());
} catch (InvocationTargetException e) {
throw new RuntimeException(e.getMessage());
}
```

把命令换成"/system/bin/cat /proc/version"可以查看linux内核版本号、gcc编译器版本号、Red Hat版本号。还比如top、ps等命令都可以使用。

另外我们还可以实现修改hosts文件、拷贝文件、在root的前提下我们还可以实现静默安装（卸载）等等，用途非常多。
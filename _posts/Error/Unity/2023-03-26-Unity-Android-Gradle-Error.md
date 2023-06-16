---
title: (Unity) Android Gradle Error
author: LHH
date: 2023-03-26 21:40 GMT+0900
categories: [Error, Unity (Error)]
tags: [Unity, Error]
---

안드로이드로 빌드하는 과정에서 생긴 에러를 해결하는 방법이다.

프로젝트 세팅을 위해 안드로이드 세팅을 진행하고 빌드한 결과 `Gradle` 관련 에러가 발생했다.

![Gradle Error](https://user-images.githubusercontent.com/110723307/227770309-7f24ba2f-0ba8-4481-bdf6-e8256d6a67de.PNG)

여러 `Gradle` 에러를 찾아봤지만 해결되는 방법은 없었고, 에러를 분석해보기로 했다.

<details>
<summary> [ Error (Click) ] </summary>
<div markdown="1">

```
FAILURE: Build failed with an exception.

* Where:
Build file 'C:\Users\82103\OneDrive\문서\Git_SourceTree\Bangchi\Bangchi\Library\Bee\Android\Prj\Mono2x\Gradle\launcher\build.gradle' line: 2

* What went wrong:
A problem occurred evaluating project ':launcher'.
> Failed to apply plugin [id 'com.android.library']
   > Your project path contains non-ASCII characters. This will most likely cause the build to fail on Windows. Please move your project to a different directory. See http://b.android.com/95744 for details. This warning can be disabled by adding the line 'android.overridePathCheck=true' to gradle.properties file in the project directory.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 4s
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8

UnityEngine.GUIUtility:ProcessEvent (int,intptr,bool&)
```

</div>
</details>

<br>

해당 에러를 분석한 결과 현재 경로에 한글이 있기 때문에 에러가 발생한 것이였다.

유니티 프로젝트 경로를 살펴보니 `문서`안에 있다는 것을 알았고 한글이 없는 경로로 이동시키니 해결이 되었다.

![succeeded](https://user-images.githubusercontent.com/110723307/227776514-7878f63e-77d2-4103-9a6d-229de5763bad.PNG)


<br>

## 💡 참고
[[안드로이드] Your project path contains non-ASCII characters 에러](https://cishome.tistory.com/70){:target="_blank"}
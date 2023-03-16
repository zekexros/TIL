# Framework에 dependency target으로 static library를 추가할 수 없는 이유

In iOS programming, a framework is a bundle that contains executable code and resources such as images, sounds, and localization files. It is designed to be shared and reused across different projects and targets. On the other hand, a static library is a compiled object code that is linked directly into an executable at compile-time.

While it is technically possible to include a static library in a framework, it is generally not recommended because it can cause problems with code duplication and versioning. When you add a static library to a target dependency, it is included in the compiled executable along with the rest of the code. This can lead to duplicate symbols if the same library is included in multiple targets, which can cause conflicts and unexpected behavior.

Frameworks, on the other hand, provide a clean separation of concerns by encapsulating their functionality and exposing a public API. This makes it easier to manage dependencies and ensures that only one version of the framework is used across all targets that depend on it.

In summary, while it is possible to include a static library in a framework, it is generally not recommended due to potential issues with code duplication and versioning. It is better to encapsulate the library's functionality in a framework and expose a public API for other targets to use.

### 요약

여러곳에서 dependency target으로 사용되는 타겟인 경우 dynamic타겟으로 만드는게 좋습니다. static은 복사하여 사용한다는 점때매 (다이나믹)프레임워크의 디펜던시에 추가되면 복제되어 충돌나고 이상하게 동작하는 에러가 발생되니 애초에 reusable하고 shared하게 사용되게끔 설계되어있는 (다이나믹)프레임워크를 사용하는 것이 좋습니다.

그러나 또 dynamic 모듈이 많아질수록 linking에 소요되는 시간으로 인한 런치스크린을 띄우는 시간이 길어지기에 용도에 맞게 잘 설계하여 섞어 사용해야 합니다.
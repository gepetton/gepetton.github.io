---
ㅇ: posts
title: "[IOS]Xcode Command LinkStoryboards failed with a nonzero exit code 에러 해결방법"
categories: IOS
---

<img src="https://github.com/gepetton/gepetton.github.io/assets/65392430/ac560568-4678-4454-a98d-316a8c46aee6" width="500" height="100">

Storyboard 분리 후 빌드하며 발생한 오류이다. 오류가 왜 발생했는지 찾아보았는데 is initial View Controller설정을 체크하지 않아서 생긴 오류이다.

<img src="https://github.com/gepetton/gepetton.github.io/assets/65392430/a0535d5a-3dfd-4770-86d4-91f0c9277fb5" width="350" height="500"><img src="https://github.com/gepetton/gepetton.github.io/assets/65392430/7a582ab8-1cda-4bc5-9d3f-d6bce99b7e40" width="350" height="500">

이렇게 체크하고 다시 빌드하면 오류가 해결된다.

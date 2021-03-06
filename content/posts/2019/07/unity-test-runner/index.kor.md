---
title: Unity 테스트 러너 도입기
date: 2019-07-12
authors: [yang.chunung]
---

안녕하세요. 플라네타리움 개발팀 양천웅입니다. 오늘은 [Unity 테스트 러너][unity-tests-runner] 도입 시도에 대한 경험담을 얘기하려 합니다.


들어가기 전에
------------

현재 저는 [Libplanet]과 Unity를 사용해 블록체인 게임 개발을 하고 있습니다. 팀에 합류한 이후 Unity로 게임을 만든다는 얘기가 나왔을 때는 막연히 'GUI 프로그래밍 경험이 없는 것도 아니고, 게임도 GUI 프로그래밍하듯이 하면 되겠지'라는 근거 없는 자신감을 가지고 시작했습니다.  
처음 프로젝트를 시작했을 때는 일정과 Unity에 대한 이해도가 부족한 상황이라 테스트를 포기하고 작업을 진행했지만, 시간이 지날수록 테스트 도입에 대한 필요성이 커졌고 그때 발견한 것이 Unity 테스트 러너였습니다.

Unity 테스트 러너는 Unity에서 자체적으로 제공하는 테스트 도구입니다. [NUnit] 기반의 테스트를 작성 후, 플레이 모드와 에디트 모드 각각에 테스트 환경을 구축하여[^1] Unity 에디터에서 테스트를 실행할 수 있습니다.


어셈블리 정의 파일 (assembly definition files)
--------------------------------------------------


처음 문서를 보며 테스트 스크립트를 만들었는데 문제를 만났습니다. 다른 라이브러리들은 정상적으로 인식이 되는데, 정작 테스트를 해야 하는 게임 코드의 네임스페이스를 인식 못 하는 것이었습니다.
원인은 Unity 에디터 실행 시 자동으로 인식되는 *Assembly-CSharp.dll*과 다르게, 테스트 스크립트 안에서는 게임 프로젝트의 스크립트를 인식 못 하기 때문이었습니다.

{{<
figure
  src="tests-asmdef.png"
  caption="<em>Tests.asmdef</em>에 의존성을 추가"
>}}

프로젝트 스크립트를 정의한 어셈블리 정의 파일을 만든 뒤 테스트용으로 정의된 어셈블리 정의 파일[^2]에 의존성을 추가해주면 문제가 해결됩니다. 

자세한 내용은 [관련 문서](https://docs.unity3d.com/kr/current/Manual/ScriptCompilationAssemblyDefinitionFiles.html)를 참고해주세요.


어셈블리 정의 파일의 플랫폼 설정
---------------------------------


어셈블리 정의 파일을 생성한 후에도 문제가 있었는데, 에디터에서는 정상적으로 작동하는 서드파티 라이브러리가 프로젝트 빌드 시에 인식이 안 되면서 깨지는 문제였습니다.
라이브러리가 Unity 에디터의 확장 기능을 제공하는 경우가 있는데, 어셈블리 정의 파일이 에디터용 기능을 빌드에 포함하려다 발생하는 문제였습니다.

{{<
figure
  src="unity-platform.png"
  caption="허용 플랫폼을 에디터만 체크"
>}}

해결 방법은 간단한데, 해당 라이브러리의 *Editor* 폴더 내부에 에디터용 정의 파일을 따로 생성 후, 플랫폼 설정의 *Include Platforms* 설정을 *Editor*만 허용하도록 변경하는 것입니다.


테스트 실행
-----------


준비가 끝났다면 이제 테스트를 작성하고 에디터에서 실행해보면 됩니다. 결과는 테스트 러너 창에서 바로 확인이 가능합니다.

{{<
figure
  src="test-result.png"
  caption="테스트 실행 결과"
>}}

현재 프로젝트에서도 아직은 테스트가 붙어있지 않은 코드가 더 많지만, Unity 테스트 러너를 도입한 이후에는 가능하면 버그 수정이나 새로운 기능 추가 시 테스트를 함께 작성하고 있으며, 덕분에 매번 게임을 실행해서 확인해야 했던 연출이나 로직 확인에 많은 시간을 줄일 수 있었습니다.  
Unity를 사용하는 다른 프로젝트에서도 기회가 될 때 도입한다면, 생산성 향상에 많은 도움이 될 것으로 생각합니다. 감사합니다!


[Libplanet]: https://github.com/planetarium/libplanet.net
[unity-tests-runner]: https://docs.unity3d.com/Manual/testing-editortestsrunner.html
[NUnit]: https://nunit.org/

[^1]: 심지어 플레이 모드 테스트는 해당 플랫폼용으로 빌드된 플레이어로 알아서 빌드 후 실행까지 가능합니다.
[^2]: 에디터에서 테스트 러너 생성 시 별도 설정을 하지 않았다면 *Assets/Tests/Test.asmdef*로 생성됩니다.

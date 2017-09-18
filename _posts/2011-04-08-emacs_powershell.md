---
layout: post
title:  괜찮은 PowerShell 용 콘솔, Emacs!
date:   2011-04-08 09:07:00 +0900
---
여러 고려끝에 제작년부터 팀의 공식 스크립트 언어와 셸 환경으로 powershell 을 사용하고 있습니다.

선정 이유는

1. vista 이상부터는 기본 설치되어 있다.

   서버급은 왠만큼 기본 설치되어 있고, 안 되어 있다고 하더라도 외부에서 다운받는 게 아니라, 
   윈도우 표준 패키지 라는 것이 유지보수에서 큰 장점입니다.

1. .net 라이브러리를 그대로 사용할 수 있다.

   보통 스크립트 언어는 기본 라이브러리가 부실;;해서, 약간 특이한 작업을 하려면
   확장 모드 같은 것을 이것저것 받아야 하는데, 버전 관리도 그렇고 번거로울 때가 많죠.
   파워셸은 .net 을 그대로 쓸 수 있으니 왠만하면 기본 라이브러리로 커버가 가능하고
   msdn 에서 c# 예제만 가지고 그대로 쓸 수 있는 (문법만 파워셸에 맞게 변경해 주면 됩니다) 장점이 있습니다.

1. windows 와 궁합이 좋다.

   이건 window 전용이다 보니 각종 window 객체에 쉽게 접근할 수 있습니다.
   cmdlet 라는 이름으로 유닉스처럼 쓸만한 콘솔용 작은 프로그램이 많은데, 유닉스보다 진보된 파이프라인(|)을
   이용해서(.net 객체를 주고받음) 굉장히 유연하게 사용할 수 있습니다.

~~~
예) 15일 전 파일들 지우는 명령
Get-childitem | where-object {$_.LastWriteTime -le [DateTime]::Now.AddDays(-15)} | Remove-Item

예) 마지막 로그파일 에서 특정 스트링을 추출
Get-childitem *.log |select-object -last 1 | get-content | select-string "ProfileLog :" 
~~~

Powershell 스크립트를 짜거나, 셸 환경에서 여러 작업을 할 때 보통은 기본 포함되어 있는 Powershell ISE 를 사용하게 됩니다. 

![PowerShell ISE]({{ site.url }}/assets/powershell_ise.jpg)
{: style="text-align: center;"}

하지만 저는 이상하게 가독성이 떨어지는 과도한 anti-aliasing 도 그렇고, 입력/출력 창이 따로 있는 것도 불편하고, 파일 열기 같은 것을 하려면 dialog 열고 하는 마우스 클릭질도 싫어서, 그냥 cmd에 powershell 로 해 놓고 콘솔용 vim 등으로 파일 편집 해 가면서 사용했습니다.

문제는 작업을 하다 보면 여러 파워셸을 뛰워놓고 싶은데, 여러 cmd 창 을 뛰우면 이동 같은 것들이 번거로울 때가 많습니다. 해서 unix 의 screen 같은 terminal multiplexer 나 tab 지원되는 터미널 같은 것을 찾게 되는데, 파워셸이 돌아가는 것이 별로 없네요..
(console2 가 훌륭하다는 평이 많지만, 안타깝게도 한글 문제가 있습니다.)

한 1년을 간간히 찾에 헤메다가, 소싯적;; 잠깐 썼던 emacs 의 파워셸 모드를 써 봤는데 아주 훌륭하네요!
여러 세션(emacs 용어로 버퍼) 을 맘대로 이동할 수 있고, 현재 작업중인 폴더에서의 파일편집도 emacs 본연의 에디팅 환경을 이용해서 키보드 만으로 여러 세션에서 뚝딱뚝딱 일을 진행할 수 있네요 ^^

![PowerShell emacs]({{ site.url }}/assets/powershell_emacs.jpg)
{: style="text-align: center;"}

예전에는 emacs 의 윈도우 버전인 ntemacs 에서도 한글 문제가 까다로웠는데(폰트가 안 보인다던지) 요즘 버전에는 그런 부분도 해결되고, 만족스럽게 쓰고 있습니다.

다만 기본 파워셸 모드는 tab completion 같은 게 잘 안 되어서 약간 설정을 해 줘야 했는데, 이 부분은 다음 번에 올리도록 해야겠네요 ^^

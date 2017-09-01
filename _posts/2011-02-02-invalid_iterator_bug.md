---
layout: post
title:  무효화된 반복자를 사용했던 버그
date:   2011-02-02 00:02:26 +0900
categories: coding
---
최근 크런치모드 로 달리는 와중에 나왔던 버그.

~~~cpp
void CUser::Func()
{
	std::vector::iterator iter;
	iter = std::find_if( m_items.begin(), m_items.end(), Item::pred );

	Item& item = *iter;
	internalMemberFunc(); // 멤버함수 안에서 m_items 의 다른 item을
                             // erase 하는 경우;;가 아주 가끔씩 있음

	item.someVariable = someValue;  // 위에서 erase 하는 경우, 
                             // item 무효화로 엉뚱한 메모리 침범
}
~~~

reference의 구현은 보통(?) 메모리 주소이다. m_items 이 담고 있는 Item 중 하나를 레퍼런스로 물고 있는 상태에서 m_items 중 하나가 erase 되면, item 레퍼런스는 실제로는 엉뚱한 메모리를 가리킬 수 있고, 이 때 item에 접근하면 메모리침범.
(잘못된 주소는 힙 상에 어느 주소인데, 크래시라도 났으면 좋았을 걸;; 지저분하게 다른 멀쩡한 힙 데이터(바로 뒤 Item)을 망가뜨리고 있었다;;)

면접에서 이렇게 짰다면 가차없이 떨어질;; 코드 이지만, 실제 함수는 저것보다 더 잘게, 더 많은 depth로 타고 들어가서 한 눈에 호출 구조를 파악하기 쉽지 않았고, 크런치 모드 특성상 변화하는 요구조건을 제대로 된 리뷰 없이 진행하다 보니 발생했다.

근본적인 문제는, state 변화를 가져오는 함수를 호출할 때에는, 함수를 호출하는 곳에서 이거 호출해도 어떤 영향이 있는지 일일이 관찰해 줘야 하는데, 어느 state 가 어떻게 변할지 파악하는 비용이 비싸다는 것(const 가 붙지 않은 이상 내부를 직접 들여다 봐야 하니). 함수형 프로그래밍 지원이 애매한 언어로는 이렇게 짜는 것이 일반적;;이다 보니……

어떻게 이런 실수를 피해 갈 수 있을지는, 사실 딱히 묘안이 있는 것은 아닌데, 그나마 들었던 생각은,

복잡한 로직은 블랙박스처럼 기계적으로 가져다 쓸 수 있도록, 함수형 언어처럼 짜자; state 변화나 side effect 를 없에거나 명시적으로 알려주고(인자로 받도록), input 이 동일하면 최대한 동일한 output 을 내도록.
( 위 경우에는, internalMemberFunc() 에서 직접 m_items 로 접근하게 하지 말고, m_items 를 레퍼런스로 인자로 넣는다던지.. )

물론 이런 방법도 극단적으로 가다 보면 문제가 있을 수 있다.
- 함수가 인자를 열 몇개 씩 주렁주렁 들고 다닐 가능성. 이렇게 되면 함수 호출을 위해 사용도 안 되는 임시변수를 만들어야 하는 경우도 생기는 등 사용성이 안 좋아진다.
- 추상적인 로직이 구체적인 state 의 수정으로 이어져야 하는 지점이 있는데, 이 지점에서부터 멤버변수를 계속 들고 다녀야 해서 그 부분이 비대해 질 수 있다. 
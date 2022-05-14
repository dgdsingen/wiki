# Architect

> 1.1. 소프트웨어 아키텍트가 하는 일
>
> 소프트웨어 아키텍트는 팀에서 독특한 위치에 있습니다. 소프트웨어 아키텍트는 프로그램 매니저가 아닙니다. 아키텍트는 소프트웨어가 언제 어떻게 전달되는지 결정하는 사람입니다. 또한 소프트웨어가 비즈니스 목표에 부합하도록 만드는 사람입니다. 코딩은 하지만 알고리즘이나 코드를 짜기보다는 더 크고 많은 것을 설계합니다. 소프트웨어 아키텍트는 여러 가지 역할에 대한 책임을 지면서, 동시에 모든 소프트웨어 개발 업무의 중심에 있는 것처럼 보이기도 합니다. 
>
> (소프트웨어 아키텍트 = [비즈니스, 기술, 사용자]의 합집합 영역에 존재)
>
> 대부분의 사람이 소프트웨어 개발 커리어를 순수하게 기술만 다루면서 시작합니다. 프로그래밍 하는 법을 익히고, 효율적인 알고리즘을 만들고, 작업 중인 소프트웨어가 완벽히 동작하도록 테스트하고, 알맞게 배포하는 일이죠. 이는 소프트웨어 아키텍트에게도 필수적인 소양입니다. 개발자에서 소프트웨어 아키텍트로 성장하려면 이에 더해 새로운 책임도 받아들여야 합니다. 
>
> 1.1.1. 엔지니어링 관점에서 문제 정의하기
>
> 소프트웨어 아키텍처를 설계하는 일은 인간중심의 설계 철학과 맥을 같이합니다. 소프트웨어와 관련된 모든 사람이 자신이 소프트웨어를 어떻게 대하고 무엇을 기대하는지 당신에게 알려줄 수 있습니다. 소프트웨어 아키텍트는 프로덕트 매니저, 프로젝트 매니저, 그리고 모든 이해관계자(stakeholder)와 협업하면서 비즈니스 목표와 요구사항을 만들어갑니다.
>
> ...
>
> 이러한 각 활동(기술 방향성 수립, 제약 설명, 아키텍처 원칙 적용)에는 완전히 다른 비기술적인 역량이 필요합니다. 종종 '소프트 스킬'(soft skill)이라고 부르는 이러한 역량은 갖추기가 꽤 어렵습니다. 이 역할에서 성공하려면 아키텍트는 기술적 아이디어를 비기술적인 용어로 설명하고, 다이어그램과 모델을 사용해 공통의 이해를 만들고, 스토리를 전달해 팀원에게 동기를 부여하고, 흥미와 관심을 북돋아주고, 팀원들에게 도전거리를 만들어주는 등 심오한 의사소통 기술을 가져야 합니다. 아키텍트로서 개발할 가치가 있는 필수적인 리더십 역량을 하나 더 말하자면 경청하는 능력입니다. 경청은 아키텍트의 지식을 높여줄 뿐만 아니라 팀과 조직이 궁극적으로 달성해야 할 기술적인 목표까지 헌신하며 달릴 수 있도록 합니다.
>
> 자신이 가진 전문적인 기술에만 집중하고 모든 기술 결정권을 쥐고 있는 아키텍트는 자신이 만든 상아탑 안에서만 살게 됩니다. 기술 리더십을 연마해 자기 자신, 팀, 조직을 잇는 튼튼한 다리를 만들어서 이러한 고립에서 벗어나야 합니다.
>
> [개발자에서 아키텍트로: 38가지 팀 활동을 활용한 실전 소프트웨어 아키텍트 훈련법]



# Architecture

- 기능/비기능 요구사항으로 나눌 경우 성능이나 보안 등의 품질 속성이 기능 요구사항으로 나타나는 부분을 가름하기 어려워진다. 그러므로 제약, 품질 속성, 기능 요구사항 영역으로 바라보자. 제약과 품질 속성은 기능 요구사항에 얼마든지 영향을 주는 요소이며 Cross-Cutting으로 존재할 수도 있다. 
- Specification은 단계별로 구분하여 진행한다. Tree 형태의 ToC처럼, Overview에서부터 점차 상세한 View로 진행해 나간다. 
- Architecture는 한번에 완벽하게 완성되어야만 하는 존재는 아니다. 점진적이고 반복적으로 Plan, Do, See를 거친다. 



# Philosophy

## Development

- 좋은 설계란 간단한것, 단순한것, 명쾌한것.
- Plan, Do, SEE
- Keep It Small & Simple.
- Focus on One thing.
- Use polymorphism. interface와 implementation을 분리해라.
- Throw an Exception rather than Pass Null.
- Don't comment bad code. Rewrite it NOW.
- 훌륭한 개발자가 되어야 훌륭한 아키텍트가 된다. 기술을 기본으로 깔고 가야 한다.
- 기술은 도구다. 기술에는 항상 목적이 있어야 한다. 기술은 결국 사람을 위한 것이다.

## Management

- 캐스팅이 반이다.
- 최선은 망치지 않는 것이다. 최대한 많이 넣은것보다, 더 넣을것이 없을때 완벽하다.
- 수직관계를 없애야한다. 그러려면 수평적 사고를 강요할게 아니라 의식을 깨부술수 있도록 구체적 방법론을 제공해야 한다. 예를 들어 팀원 전체가 반말을 한다던가.
- 핵심 제품 70%, 신규 제품 20%, 장기 프로젝트 10%
- 전문가답게 문제를 해결해주고, 간단하게 설명해라.
- 의사결정을 하기 전에 현장으로 가라. 직접 보지 않으면 보고자의 주관에 흔들리게 된다.
- 점진적이고, 반복적으로.
- 자리가 바뀌면, 역할도 바뀌어야 한다.
- 부하가 아닌 파트너가 되라.
- 일에서는 우정이 아닌 신뢰를 쌓아라.
- 의제를 설정해라. 의제 설정의 근거는 3가지다.
    - 어디에 있는가.
    - 어디로 가는가.
    - 어떻게 가는가.

## Attitude

- 서비스 분야에서 처음 일하게 된다면 아래 사항들에 대해 진지하게 고민해 볼 필요가 있다.
- 요구사항이 없으면 일을 안한다.
    - 아웃소싱 프로젝트는 대부분 목표 시스템과 요구사항이 명확합니다.
    - 그래서 요구사항 없이 일을 하게 되면 사고로 이어지는 경우가 많습니다.
    - 그렇게 일하도록 배웠고 그래야만 책임 분쟁에서 자유로울 수 있습니다.
- 서비스 및 사업모델의 문제점을 지적하기만 합니다.
    - 기술 구현만 할 생각이기 때문에 요구 사항에 허점이 있으면 안됩니다.
    - 데이터 오류를 피하기 위해 문제점을 미리 찾아서 보고해 줍니다.
    - 그렇게 일하도록 배웠기 때문에 문제를 알아서 해결하려 하지 않습니다.
- 사업과 서비스 업무에 전혀 관여하지 않습니다.
    - 제 경험상 사업 및 서비스 정책에 대해 가장 현실적으로 조언을 해 줄 수 있는 사람은 개발자들입니다.
    - 왜냐하면 자기 손으로 직접 기능을 구현하기 때문에 부족한 점을 가장 먼저 보게 됩니다.
    - 그리고 데이터가 잘못 쌓이는 걸 가장 먼저 알게 됩니다.
    - 하지만, 운영 경험이 없는 개발자들은 그런 것에 대해 거의 신경쓰지 않습니다.
- 완벽하게 설계하고 변경을 고려하지 않습니다.
    - 운영을 해보지 않은 개발자는 변경에 대해 극도로 보수적입니다.
    - 왜 변경이 발생해야 하는지 이해하지 못합니다.
    - 완벽한 설계 속에서 문제를 해결하려고 합니다.
    - 예외를 인정하지 않습니다.
    - 그래서 사업이 때를 놓치는 경우가 적지 않습니다.
- 데이터 자산에 대해서 고려하지 않습니다.
    - 일반적으로 SI 프로젝트는 기능을 구현해 주고는 철수합니다.
    - 따라서 데이터를 어떻게 활용할 지에 대해서는 고민하지 않습니다.
    - 그래서 SI 개발자들은 데이터 오류검증이나 활용 방안에 대해서는 전혀 관심이 없습니다.
    - 하지만, 서비스에서 데이터는 시장을 나타내는 지표이며, 사업의 목적이기도 합니다.
    - 데이터에 관심이 없는 개발자를 만나게 되면, 반드시 어느 부분에서는 펑크가 나게 됩니다.
- 유지보수 업무를 단순 운영업무로 이해합니다.
    - 일반적인 IT 아웃소싱은 시스템을 구매하는 것입니다.
    - 기능이 변경되면 별도의 예산을 통해 구매합니다.
    - 따라서 유지보수란 잘못된 기능을 보수하는 수준에 그칩니다.
- 그래서 개발하면서 운영하는 것에 대해 잘 이해하지 못합니다.버전관리나 형상관리, 배포관리의 중요성을 이해하지 못하는 경우가 많습니다.
- 서비스에 참여하게 되면 위 사항과 거의 반대로 생각하고 행동해야 합니다. 개발자도 사용자 가치를 만들어 내는 직접적인 주체이기 때문입니다.
- SI 개발자라도 훈련을 통해 이런 주체성을 연습할 수 있습니다. 바로 자신만의 개인 프로젝트를 해보는 것입니다.
- 그러면 ‘의사결정’과 ‘기술의 적정성’ 등에 대해 생각해볼 기회가 많아지게 됩니다. 태도라고 표현되는데 기술보다 생각과 마음을 바꿔야 한다는 것이 더 정확한 것 같습니다.
- 요약하면
    - 개인 프로젝트를 하자.
    - 기술 외에 취미 하나씩 가지자. (기술을 공부하게 만드는 동기)
    - 주도적이고 주체성 있는 개발자가 되자.
    - 적정기술과 모듈별 재개발에 익숙해지자.

## Pragmatic Programmer

- 자신의 기술craft에 관심과 애정을 가져라.
    - 소프트웨어 개발을 잘 해보려는 생각이 없다면 왜 인생을 그 일을 하면서 보내는가?
- 자신의 일에 대해 생각하면서 일하라!
    - 자동 조종 장치를 끄고 직접 조종하라. 스스로의 작업을 지속적으로 비판하고 평가하라.
- 어설픈 변명을 만들지 말고 대안을 제시하라.
    - 변명하는 대신 대안을 제시하라. 그 일은 할 수 없다고 말하지 말고, 무엇을 할 수 있는지에 대해 설명하라.
- 깨진 창문을 내버려두지 말라.
    - 눈에 뜨일 때마다 나쁜 설계, 잘못된 결정, 좋지 않은 코드를 고쳐라.
- 변화의 촉매가 되라.
    - 사람들에게 변화를 강요할 수는 없다. 대신, 미래가 어떤 모습일지 그들에게 보여주고 미래를 만드는 일에 그들이 참여하도록 도우라.
- 큰 그림을 기억하라.
    - 주변에 무슨 일이 일어나는지 점검하는 일을 잊어버릴 정도로 세부사항에 빠지지 말라.
- 품질을 요구사항으로 만들어라.
    - 프로젝트의 진짜 품질 요구사항을 결정하는 자리에 사용자를 참여시켜라.
- 지식 포트폴리오에 주기적으로 투자하라.
    - 학습을 습관으로 만들어라.
- 읽고 듣는 것을 비판적으로 분석하라.
    - 벤더, 매체들의 야단법석, 도그마에 흔들리지 말라. 여러분과 여러분 프로젝트의 관점에서 정보를 분석하라.
- 무엇을 말하는가와 어떻게 말하는가 모두 중요하다.
    - 효과적으로 전달하지 못한다면 좋은 생각이 있어봐야 소용없다.
- DRY(Don’t Repeat Yourself)-반복하지 마라
    - 어떤 지식 한 조각도 하나의 시스템 안에서는 모호하지 않고, 권위 있고, 단 하나뿐인 표현을 가져야 한다.
- 재사용하기 쉽게 만들라.
    - 재사용하기 쉽다면, 사람들이 재사용할 것이다. 재사용을 촉진하는 환경을 만들라.
- 관련 없는 것들 간에 서로 영향이 없도록 하라.
    - 컴포넌트를 자족적이고, 독립적이며, 단 하나의 잘 정의된 목적만 갖도록 설계하라.
- 최종 결정이란 없다.
    - 돌에 새겨진 것처럼 불변하는 결정은 없다. 그렇게 생각하는 대신, 모든 결정이 해변의 백사장 위에 쓴 글자와 같다고 생각하고 변화에 대비하라.
- 목표물을 찾기 위해 예광탄을 써라.
    - 예광탄은 이것저것을 시도해보고 그것들이 목표와 얼마나 가까운 데 떨어지는지 보는 방법으로 목표를 정확히 맞추게 해준다.
- 프로토타입을 통해 학습하라.
    - 프로토타이핑은 배움의 경험이다. 프로토타이핑의 가치는 만들어낸 코드에 있지 않고, 여러분이 배운 교훈에 있다.
- 문제 도메인에 가깝게 프로그래밍하라.
    - 사용자의 언어를 사용해서 설계와 코딩을 하라.
- 추정을 통해 놀람을 피하라.
    - 시작하기 전에 추정부터 하라. 잠재적인 문제점들을 미리 찾아내게 될 것이다.
- 코드와 함께 일정도 반복하며 조정하라.
    - 구현하면서 얻는 경험을 이용해서 프로젝트의 시간 척도를 세밀히 조정하라.
- 지식을 일반 텍스트로 저장하라.
    - 일반 텍스트 형식은 시일이 지났다고 못쓰게 되는 일이 없다. 일반 텍스트 형식은 여러분의 작업을 활용하고 디버깅과 테스팅을 쉽게 만드는 데 도움이 된다.
- 명령어 셸의 힘을 사용하라.
    - 그래픽 사용자 인터페이스로는 할 수 없는 일에 셸을 이용하라.
- 하나의 에디터를 잘 사용하라.
    - 에디터를 마치 손의 연장延長으로 자유자재로 다루어야 한다. 여러분이 사용하는 에디터는 설정을 바꿀 수 있고, 확장가능하고, 프로그램 가능해야 한다.
- 언제나 소스코드 관리 시스템을 사용하라.
    - 소스코드 관리는 여러분 작업을 위한 타임머신이다. 언제라도 과거로 돌아갈 수 있게 해준다.
- 비난 대신 문제를 해결하라.
    - 버그가 여러분 잘못인지 다른 사람 잘못인지는 별로 중요하지 않다. 그것은 여전히 여러분의 문제이며, 여전히 고쳐야 할 필요가 있다.
- 디버깅을 할 때 당황하지 마라.
    - 숨을 깊게 들이 쉬고, 무엇이 버그를 일으키는지 ‘생각하라!’
- ‘select’는 망가지지 않았다.
    - OS나 컴파일러의 버그를 발견하는 일는 정말 드물게 일어나며, 심지어 써드파티 제품이나 라이브러리일지라도 드문 일이다. 버그는 애플리케이션에 있을 가능성이 가장 크다.
- 가정하지 마라. 증명하라.
    - 진짜 데이터와 경계 조건이 있는 실제 환경에서 여러분이 내렸던 가정들을 증명하라.
- 텍스트 처리 언어를 하나 익혀라.
    - 여러분은 하루 가운데 많은 시간을 텍스트와 씨름하며 보낸다. 왜 여러분 대신 컴퓨터가 그 일의 일부를 하게끔 만들지 않는가?
- 코드를 작성하는 코드를 작성하라.
    - 코드 생성기는 생산성을 증가시키며 중복을 막는 일에도 도움이 된다.
- 완벽한 소프트웨어는 만들 수 없다.
    - 소프트웨어는 완벽할 수 없다. 피할 수 없이 나타나는 에러로부터 여러분의 코드와 사용자들을 보호하라.
- 계약에 따른 설계를 하라.
    - 코드가 실제로 하기로 한 것을 문서화하고 검증하기 위해 계약을 사용하라.
- 일찍 작동을 멈추게 하라.
    - 보통은 죽은 프로그램이 절름발이 프로그램보다 해를 훨씬 덜 끼친다.
- 단정문을 사용해서 불가능한 상황을 예방하라.
    - 단정은 여러분이 세운 가정을 검증해준다. 확실한 것이 없는 세상에서 여러분의 코드를 보호하려면 단정문을 사용하라.
- 예외는 예외적인 문제에 사용하라.
    - 예외를 잘못 쓰면 고전적 스파게티 코드의 모든 가독성과 유지보수 문제를 그대로 겪을지도 모른다. 예외는 예외적인 일들만을 위해 남겨두어라.
- 시작한 것은 끝내라.
    - 가능하다면, 리소스를 할당한 루틴이나 객체가 해제도 책임져야 한다.
- 모듈간의 결합도를 최소화하라.
    - 디미터 법칙을 적용하고‘부끄럼 타는shy’ 코드를 작성해서 결합이 생기는 일을 피하라.
- 통합하지 말고 설정하라.
    - 애플리케이션에서 기술 선택을 설정 옵션으로 구현하고, 통합하거나 만들어 넣지 말라.
- 코드에는 추상화를, 메타데이터에는 세부 내용을.
    - 프로그램은 최대한 일반화해서 만들고, 세부 사항들은 가능하면 컴파일된 코드 기반 바깥으로 빼라.
- 작업흐름 분석을 통해 동시성을 개선하라.
    - 사용자의 작업흐름이 허용하는 동시성을 최대한 활용하라.
- 서비스를 사용해서 설계하라.
    - 서비스, 곧 잘 정의되고 일관성 있는 인터페이스를 통해 의사소통하는 독립적이고 동시성 있는 객체들의 관점에서 설계하라.
- 언제나 동시성을 고려해 설계하라.
    - 동시성이 가능하도록 설계하면, 더 적은 가정만 내리고서도 더 깔끔한 설계를 할 수 있다.
- 모델에서 뷰를 분리하라.
    - 애플리케이션을 모델과 뷰의 관점으로 설계해서 적은 비용만 들이고도 유연함을 얻어내라.
- 칠판을 사용해 작업흐름을 조율하라.
    - 참여하는 요소들의 독립성independence과 고립성isolation을 유지하면서도 개별적인 사실과 에이전트를 잘 조율하려면 칠판을 사용하라.
- 우연에 맡기는 프로그래밍을 하지 말라.
    - 정말 믿을 만한 것만 믿어야 한다. 우발적인 복잡함을 조심하고, 우연한 행운을 목적의식을 가지고 만든 계획과 착각하지 말라.
- 여러분의 알고리즘의 차수를 추정하라.
    - 코드를 작성하기 전에, 실행 시간이 대략 얼마나 걸릴지 감을 잡아 놓아라.
- 여러분의 추정을 테스트하라.
    - 알고리즘의 수학적 분석이 모든 것을 다 알려주지는 않는다. 실제 대상 환경에서 코드의 수행 시간을 측정해보라.
- 일찍 리팩터링하고, 자주 리팩터링하라.
    - 정원의 잡초를 뽑고 식물 배치를 조정하는 것과 똑같이, 코드도 필요할 때면 언제라도 다시 작성하고 다시 작업하고 다시 아키텍처를 만들라. 문제의 근원을 해결하라.
- 테스트를 염두에 두고 설계하라.
    - 코드를 한 줄이라도 쓰기 전에 테스팅에 대해 생각하기 시작해야 한다.
- 소프트웨어를 테스트하라. 그렇지 않으면 사용자가 테스트하게 될 것이다.
    - 가차 없이 테스트하라. 사용자가 여러분을 위해 버그를 찾게 만들지 말라.
- 자신이 이해하지 못하는, 마법사가 만들어준 코드는 사용하지 말라.
    - 마법사는 엄청난 양의 코드를 만들 수 있다. 그것들을 프로젝트에 통합해 넣기 전에 그 코드 내용을 전부 이해하는지 확실히 해놓도록 하라.
- 요구사항을 수집하지 말고 채굴하라.
    - 요구사항이 지면에 놓여져 있는 경우는 퍽 드물다. 보통은 가정과 오해, 정치政治의 지층들 속 깊이 묻혀 있다.
- 사용자처럼 생각하기 위해 사용자와 함께 일하라.
    - 시스템이 정말로 어떻게 사용될지 통찰력을 얻을 수 있는 가장 좋은 방법이다.
- 구체적인 것보다 추상적인 것이 더 오래간다.
    - 구현 말고 추상에 투자하라. 추상은 서로 다른 구현이나 새로운 기술의 출현 때문에 빗발치듯 생기는 변화를 견뎌내고 살아남을 수 있다.
- 프로젝트 용어사전을 사용하라.
    - 프로젝트에서 쓰이는 특정 용어와 어휘들의 유일한 출처를 만들고 유지하라.
- 생각의틀을벗어나지말고, 틀을찾아라.
    - 해결이 불가능해 보이는 문제와 마주쳤을 때, 진짜 제약 조건을 찾아라. 스스로에게 이렇게 물어보라. ‘정말로 반드시 이런 방식으로 해야 하는 일인가? 꼭 해야만 하는 일이긴 한 건가?’
- 준비가 되었을 때 시작하라.
    - 여러분은 살아오면서 경험을 쌓아왔다. 자꾸 거슬리는 의혹을 무시하지 말라.
- 어떤 일들은 설명하기보다 실제로 하는 것이 더 쉽다.
    - 명세의 나선에 빠지지 말라. 언젠가는 코딩을 시작해야 한다.
- 형식적 방법의 노예가 되지 마라.
    - 여러분의 개발 실천방법과 개발 능력의 맥락 안에 넣어보지 않고, 맹목적으로 어떤 기법을 채택하지 말라.
- 비싼 도구가 더 좋은 설계를 낳지는 않는다.
    - 벤더들의 과장, 어떤 분야의 도그마 그리고 가격표의 휘광에 넘어가지 말라. 도구 자체의 장점만 갖고 판단하라.
- 팀을 기능 중심으로 조직하라.
    - 설계자와 코더를, 테스트 담당자와 데이터 모델 담당자를 분리시키지 말라. 코드를 만드는 방식에 맞춰 팀을 만들어라.
- 수작업 절차를 사용하지 말라.
    - 셸 스크립트나 배치 파일은 똑같은 명령을, 똑같은 순서로, 어느 때라도 반복해서 실행해준다.
- 일찍 테스트하고, 자주 테스트하라. 자동으로 테스트하라.
    - 매번 빌드할 때마다 실행되는 테스트가 책꽂이의 테스트 계획보다 훨씬 효과적이다.
- 모든 테스트가 통과하기 전엔 코딩이 다 된게 아니다.
    - 뭐 더 할 말 있나?
- 파괴자를 써서 테스트를 테스트하라.
    - 코드의 별도 복사본을 만들고, 그 복사본에 고의로 버그를 넣은 다음 테스트가 잡아내는지 검증하라.
- 코드 커버리지보다 상태 커버리지를 테스트하라.
    - 중요한 프로그램 상태들을 파악해서 테스트하라. 단지 많은 코드 줄 수를 테스트 범위 안에 넣는 것만으로는 충분하지 않다.
- 버그는 한 번만 잡아라.
    - 인간 테스터가 버그를 찾아내면, 그 때가 인간 테스터가 그 버그를 찾는 마지막 순간이 되어야 한다. 그 순간 이후부터는 자동화된 테스트가 그 버그를 담당하도록 만들라.
- 한국어도 하나의 프로그래밍 언어인 것처럼 다루라.
    - 코드를 작성하는 것처럼 문서도 작성하라. DRY 원칙을 존중하고, 메타데이터를 사용하고, MVC 모델을 쓰고, 자동 생성을 이용하고 등등.
- 문서document가 애초부터 전체의 일부가 되게하고, 나중에 집어넣으려고 하지 말라.
    - 코드와 떨어져서 만든 문서가 정확하거나 최신 정보를 반영하기는 더 힘들다.
- 사용자의 기대를 부드럽게 넘어서라.
    - 사용자들이 무엇을 기대하는지 이해한 다음, 그것보다 약간 더 좋은 것을 제공하라.
- 자신의 작품에 서명하라.
    - 옛날 장인들은 자신의 작업 결과물에 서명하는 일을 자랑스럽게 여겼다. 여러분도 마찬가지여야 한다.

## Manifesto for Agile Software Development

We are uncovering better ways of developing software by doing it and helping others do it. Through this work we have come to value:

- Individuals and interactions over processes and tools
- Working software over comprehensive documentation
- Customer collaboration over contract negotiation
- Responding to change over following a plan

That is, while there is value in the items on the right, we value the items on the left more.

### Principles behind the Agile Manifesto

We follow these principles:

- Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.
- Welcome changing requirements, even late in development. Agile processes harness change for the customer's competitive advantage.
- Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.
- Business people and developers must work together daily throughout the project.
- Build projects around motivated individuals. Give them the environment and support they need, and trust them to get the job done.
- The most efficient and effective method of conveying information to and within a development team is face-to-face conversation.
- Working software is the primary measure of progress.
- Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.
- Continuous attention to technical excellence and good design enhances agility.
- Simplicity--the art of maximizing the amount of work not done--is essential.
- The best architectures, requirements, and designs emerge from self-organizing teams.
- At regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly.



# References

- [기술자의 히포크라테스 선서](https://blog.fupfin.com/?p=188) 
- [draw.io](https://www.draw.io/) 
- [Microsoft Learn](https://docs.microsoft.com/ko-kr/learn/) 
- [Google API Improvement Proposals](https://google.aip.dev/) 
- [microservices.io](http://microservices.io/) 
- [조대협 : Micro Service Architecture](http://bcho.tistory.com/tag/MSA) 
- [Circuit Breaker Pattern](http://egloos.zum.com/pulgrims/v/3047353) 
- [Hystrix](https://github.com/Netflix/Hystrix/wiki) 
- [BFF : Backend For Frontend](https://www.google.co.kr/search?ei=N6TtWdnzLceA8gXswKr4Cw&q=bff+%ED%8C%A8%ED%84%B4&oq=bff+%ED%8C%A8%ED%84%B4&gs_l=mobile-gws-serp.3...2603.4917.0.5308.7.7.0.0.0.0.398.1774.2-3j3.6.0....0...1.1j4.64.mobile-gws-serp..2.4.1117...0j30i10k1.200.xG85uUX9M8A) 
- [PAYCO 쇼핑 마이크로서비스 아키텍처 전환기](https://www.joinc.co.kr/w/man/12/msaPayco?utm_source=gaerae.com&utm_campaign=%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%8A%A4%EB%9F%BD%EB%8B%A4&utm_medium=social) 
- [뱅크샐러드 Monolithic to MSA 사례](https://blog.banksalad.com/tech/how-banksalald-decomposes-legacy-services/) 
- [IBM Cloud Architecture & Solution Engineering](https://github.com/ibm-cloud-architecture) 
- [ArchUnit - UnitTest로 아키텍처 검사를](https://d2.naver.com/helloworld/9222129) 

## Capacity
- [Tech stack rebuild for a new Facebook.com](https://engineering.fb.com/web/facebook-redesign/) 
- [네이버 메인 페이지의 트래픽 처리](https://d2.naver.com/helloworld/6070967) 
- [대용량 세션을 위한 로드밸런서](https://d2.naver.com/helloworld/605418) 
- [L4/L7 스위치의 대안, 오픈 소스 로드 밸런서 HAProxy](https://d2.naver.com/helloworld/284659) 
- [HAProxy - The Reliable, High Performance TCP/HTTP Load Balancer](http://www.haproxy.org/) 
- [Apache mod_proxy를 이용한 reverse proxy와 로드밸런싱](https://khlee03.tistory.com/entry/Apache-modproxy%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-reverse-proxy%EC%99%80-load-balancing) 
- [NginX를 이용한 static 컨텐츠 서비스 와 캐시 설정](https://www.joinc.co.kr/w/man/12/nginx/static) 
- [Reverse Proxy with Caching | NGINX](https://www.nginx.com/resources/wiki/start/topics/examples/reverseproxycachingexample/) 
- [A Guide to Caching with NGINX and NGINX Plus - NGINX](https://www.nginx.com/blog/nginx-caching-guide/) 
- [천만 명의 사용자에게 1분 내로 알림 보내기 (병렬프로세스의 최적화)](https://taetaetae.github.io/2019/01/02/faster-parallel-processes/index.html) 
- [신규 포인트 시스템 전환기 #1 - 개발 단계 - 우아한형제들 기술 블로그](http://woowabros.github.io/experience/2018/10/12/new_point_story_1.html) 
- [신규 포인트 시스템 전환기 #2 - 오픈 준비 단계 - 우아한형제들 기술 블로그](http://woowabros.github.io/experience/2018/10/15/new_point_story_2.html) 
- [비트윈의 멀티티어 아키텍처를 위한 프레젠터 이야기](http://engineering.vcnc.co.kr/2015/11/presenter-multitier-architecture/) 
- [타다 시스템 아키텍처](http://engineering.vcnc.co.kr/2019/01/tada-system-architecture/) 
- [Sharding Platform](https://d2.naver.com/helloworld/14822) 
- [Sharding과 Partitioning의 차이점](http://theeye.pe.kr/archives/1917) 
- [Database의 샤딩(Sharding)이란?](https://nesoy.github.io/articles/2018-05/Database-Shard) 
- [대용량 시스템을 위한 데이타베이스 아키텍쳐-Sharding &amp; Query Off Loading](https://bcho.tistory.com/670) 
- [MySQL High Availability at GitHub](https://githubengineering.com/mysql-high-availability-at-github/) 
- [Orchestrator at GitHub](https://githubengineering.com/orchestrator-github/) 
- [github/orchestrator: MySQL replication topology management and HA](https://github.com/github/orchestrator) 
- [SSD는 소프트웨어 아키텍처를 어떻게 바꾸고 있는가?](https://d2.naver.com/helloworld/162498) 
- [Wikipedia System Architecture](https://upload.wikimedia.org/wikipedia/commons/4/4f/Wikimedia-servers-2009-04-05.svg) 
- [Shopify 아키텍처의 진화](https://blog.gaerae.com/2019/02/evolution-of-shopifys-architecture.html) 
- [분산 코디네이터 Zookeeper 소개](https://bcho.tistory.com/1016) 

## Logging
- [Logback을 활용한 Remote Logging](http://www.nextree.co.kr/p5584/) 
- [대규로 로그 서비스 1](https://www.slideshare.net/ssuser380e9c/ndc18-95524337) 
- [대규모 로그 서비스 2](https://www.slideshare.net/ssuser380e9c/ndc18-2-95522893) 
- [웹상에 퍼져있는 slf4j 연동관련 정리](http://areumgury.blogspot.com/2016/02/slf4j.html) 
- [Hadoop과 MongoDB를 이용한 로그분석시스템](https://d2.naver.com/helloworld/1016) 
- [GitHub - sonegy/how-to-use-logback: 로그백 사용법](https://github.com/sonegy/how-to-use-logback) 
- [로그 시스템 #1 - 자바 로그 프레임웍 종류](https://bcho.tistory.com/1312) 
- [로그 시스템 #2 - 기본 로깅 및 JSON 포맷으로 로깅하기](https://bcho.tistory.com/1313) 
- [로그 시스템 #3 - JSON 로그에 필드 추가하기](https://bcho.tistory.com/1314) 
- [로그 시스템 #4 - MDC를 이용하여 쓰레드별로 로그 분류하기](https://bcho.tistory.com/1316?category=431297) 
- [로그 시스템 #5 - Spring boot에서 JSON 포맷 로깅과 MDC 사용하기](https://bcho.tistory.com/1317) 
- [로그 시스템 #6 - Spring Boot에서 Zipkin을 이용한 분산 시스템 로깅](https://bcho.tistory.com/m/1319) 
- [로그 시스템 #7 - 스택드라이버로 로그 백앤드 구축하기](https://bcho.tistory.com/m/1321) 


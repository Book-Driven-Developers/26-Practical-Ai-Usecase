## 강의 진행 방식
- 레거시 코드 존재
- 레거시 리팩토링 및 추가 요구사항을 AI를 통해 구현
- ai가 참고할 수 있는 guideline.md를 관리하며, 수행할 때마다 이를 보완
- 부족한 코드를 직접 개선

## 강의를 통해 얻어가고 싶었던 것
- ai를 '잘' 활용하여 최대한 수정사항을 적게 만들어가는 방법
- 물론 수정사항이 없을 순 없지만, 그 간극을 줄여나가는 방법
- ai를 통해 프로젝트를 만드는 방법

## LLM 활용 방법 트렌드(?)
- 예전에는 프롬프트 엔지니어링이 중요했지만, 이제는 프롬프팅은 당연한 것이고 어떻게 컨텍스트를 잘 관리하냐가 제일 중요하다.
- 결국 강의에서처럼 특별한 기능 없이 guideline.md를 계속 보완해 나가면 llm이 인지해야 하는 컨텍스트가 늘어나기 때문에 토큰이 많이 소비되고, 정확도는 떨어질 것이다.
- 여러 llm이 있지만 컨텍스트 관리를 가장 잘 도와주는 llm은 클로드 코드가 가장 선두에 있다.  

## 클로드 코드
- mcp -> skills -> sub agents -> agent team
- [skiils](https://code.claude.com/docs/ko/skills): 효율적인 컨텍스트 관리를 위해, 필요한 경우에만 skills.md를 컨텍스트에 로드하는 기능
- [sub agents](https://code.claude.com/docs/ko/settings#subagent-%EA%B5%AC%EC%84%B1)
  - 여러 subagent가 동시에 독립적으로 작업 가능
  - 각각의 subagent마다 제한 토큰이 할당되기 때문에, 한 번에 요청으로 해결되지 않던 이슈 해결
- agent team
  - ...

## 그래서 무엇을 해야 할까?
- 클로드 코드 docs 읽고 사용법 숙지하기
- 다른 사람들의 활용법 참고 및 적용 후 서로 공유하기
- 나는 어떻게 활용하고 있는 지 공유하기

## 참고
- [빌더조쉬 - 커서맛피아 인터뷰](https://www.youtube.com/watch?v=GL3LXWBZfy0)
  - 제가 유입된 경로.. 
- [커서맛피아님의 클로드 코드 활용 방법](https://www.youtube.com/live/wDP91skrk5M)
- [위 유튜브 영상에 대한 ai 정리본](https://lilys.ai/digest/7314484/7894470?s=1&noteVersionId=4315707)

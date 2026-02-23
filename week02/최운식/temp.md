클로드 코드 사용

클로드 코드를 사람들이 더 하드하게 사용하기 위해서 open-code, oh-my-opencode 등 외부 플러그인 기능들이 나왔다.
- 이들은 오케스트레이션 기능과 같이 최상위에 관리자 agent가 존재하고 계획, 구현, 검증 등을 반복하고 이때 각 부분에 대해 적합한
ai 모델들을 여러개 사용하는 방식이였음.
- 다만 클로드, 제미나이에서는 이러한 방식을 막음.
- 이후 agent, agent team 기능들이 출시 되었다.

로컬이나 프로젝트에서도 agent를 상황에 맞게 여러개 등록해 놓고 사용하게 된다.
- settings.json에 "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" 을 등록하면 agent team 기능을 사용할 수 있다.
- 에이전트 팀 생성을 자연어로 클로드 코드로 요청하면 생성이 된다.
- 그러면 open-code 처럼 리드와 팔로워 구조로 agent team이 생성되고 병렬적으로 작업이 수행된다.

agent 등록
- name: spring-boot-senior-dev
- name: test-code
- ...

위와 같이 여러개의 agent를 등록할 수 있다.
작업 요청 시 적절한 agent가 선택되어 작업이 수행된다.


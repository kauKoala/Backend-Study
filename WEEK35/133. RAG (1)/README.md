# RAG (1)

```markdown
<시스템 프롬프트>
너는 친절한 답변을 하는 AI 어시스턴트야.
다음의 질문에 답변해줘.

<사용자 입력(질문)>
삼성전자에서 개발한 AI의 이름은?
```

위와 같이 LLM API를 호출하면 환각 현상(Hallucination) 발생, **왜?**

⇒ outdated된 지식만 가지고 있다면, 정확한 답변을 주지 못함.

**해결책**

: 참고할 정보를 줘서 정확한 답변을 하도록 함 (pdf 문서 등의 정보를 함께 줌)

---

## **1. RAG란?**

Retrieval-Augmented Generation

사전에 저장해 둔 풍부한 데이터(=context)를 참조하는 프로세스

- LLM은 최신 정보를 가지고 있지 않을 확률이 있음 (최신 정보를 넣어둘 수 있음)
- 전문 지식의 경우, LLM은 정확한 정보를 주지 않을 수 있음

**LLM에 너무 많은 RAG 데이터를 제공하면 안되는 이유는?** 

(= 몇백장의 pdf를 제공하면 안되는 이유)

1. input이 클수록 비용이 크다. 
2. lost in the middle

## 2. pre-processing

![image](https://github.com/user-attachments/assets/59fb0338-487e-42e9-8292-1d81255c0d52)
![image](https://github.com/user-attachments/assets/2d31d22d-f42e-4520-84a8-2a72f63245a6)


### Document Load

LangChain 등을 활용하면, 문서 형식에 구애받지 않고 다양한 데이터 로드 가능

(cf. LangChain이란? LLM(예: GPT) 기반의 애플리케이션을 더 쉽게 만들 수 있게 도와주는 파이썬/자바스크립트 프레임워크**)**

### Text Split

token 수 기반의 chunk size를 지정해 text를 split함

- 해당 chunk(=단락) 단위로 저장함
    
    ![image](https://github.com/user-attachments/assets/178be077-eb32-4740-add1-9921f540f213)

- 모든 단락 대상으로 유사도 검사를 함
- 단락을 몇 개 사용할 것인지 **k**를 잘 지정하는 것이 중요함

cf. **chunk overlab**

![image](https://github.com/user-attachments/assets/dcec5454-1037-496d-96be-b5a6eefe99e3)

; 문장을 겹쳐서 split하여, 중간에 잘리지 않도록 조정

### Embed

**임베딩**이란? 문장을 수치 표현으로 변환하는 것

e.g. 매운 맛; 0.1 / 새콤한 맛; 0.9 ⇒ 매콤새콤한 맛; 0.5

질문도 임베딩하여 수치화하고, 단락(=chunk)도 임베딩하여 수치화해서 해당 수치를 비교하여 유사도 체크

⇒ 단일 수치로는 불가능하므로, 벡터(vector) 수치로 표현됨 

(cf. openAI 임베딩 - 1,536차원을 사용함)

임베딩할 때도 돈이 듦.

답변을 받는 것만큼 비싸진 않지만, 다량의 문서를 임베딩할 때는 돈이 많이 들 수 있음

; 프로그램 실행할 때마다 임베딩을 하는 것이 아니라, 문서 load → split → embed한 후 store하는 것이 정석 (= vector store)

여기까지가 **사전 단계**

RAG(2)는 **실행 단계(= Runtime)**를 다룰 예정

### 참고자료

[EP01. #RAG 의 동작 과정 쉽게 이해하기!](https://www.youtube.com/watch?v=zybyszetEcE)

- chatGPT 4o

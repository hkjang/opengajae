# Node-RED Tool Extension

Node-RED 플로우를 프로그래밍 방식으로 관리하고 생성하기 위한 OpenClaw 확장입니다.

## 설치 및 설정

```yaml
# openclaw/config.yaml
plugins:
  node-red-tool:
    baseUrl: "http://localhost:1880" # Node-RED 서버 URL
    token: "your-token" # adminAuth 사용 시 토큰
    deploymentType: "flows" # full, nodes, flows, reload
    readOnly: false # true면 조회만 허용
```

## Actions 개요

| 카테고리        | Action            | 설명                       |
| --------------- | ----------------- | -------------------------- |
| **API 관리**    | `flows_get`       | 전체 플로우 조회           |
|                 | `flows_deploy`    | 플로우 배포                |
|                 | `flows_state_get` | 플로우 상태 조회           |
| **플로우 관리** | `flow_add`        | 새 플로우 탭 추가 (서버)   |
|                 | `flow_update`     | 플로우 업데이트 (서버)     |
|                 | `flow_create`     | 플로우 탭 생성 (로컬)      |
| **노드 관리**   | `nodes_list`      | 설치된 노드 목록           |
|                 | `nodes_install`   | 노드 모듈 설치             |
|                 | `node_create`     | 노드 생성 (로컬)           |
|                 | `nodes_connect`   | 노드 연결 (로컬)           |
| **패턴**        | `pattern_build`   | 플로우 패턴 빌드           |
|                 | `node_types`      | 사용 가능한 노드 타입 조회 |
| **템플릿**      | `templates_list`  | 템플릿 목록 조회           |
|                 | `template_apply`  | 템플릿 적용                |
| **도우미**      | `catalog_search`  | 노드 카탈로그 검색         |
|                 | `catalog_info`    | 노드 상세 정보             |
|                 | `flow_validate`   | 플로우 검증                |
|                 | `flow_analyze`    | 플로우 분석                |

---

## 플로우 패턴 (pattern_build)

6가지 내장 패턴으로 빠르게 플로우를 생성할 수 있습니다.

### simple - 기본 플로우

```
inject → function → debug
```

```json
{
  "action": "pattern_build",
  "patternType": "simple",
  "label": "My Simple Flow",
  "handlerFunc": "msg.payload = msg.payload.toUpperCase();\nreturn msg;",
  "interval": "60"
}
```

### http-api - HTTP API 엔드포인트

```
http in → handler → http response
```

```json
{
  "action": "pattern_build",
  "patternType": "http-api",
  "label": "User API",
  "baseUrl": "/api/users",
  "method": "get",
  "handlerFunc": "msg.payload = { users: [] };\nreturn msg;"
}
```

### switch - 조건 분기

```
input → switch → [output1, output2, ..., else]
```

```json
{
  "action": "pattern_build",
  "patternType": "switch",
  "label": "Status Router",
  "properties": { "property": "payload.status" },
  "conditions": [
    { "value": "success" },
    { "value": "error" },
    { "value": "pending" }
  ]
}
```

### error-handler - 에러 처리

```
catch → handler → debug
```

```json
{
  "action": "pattern_build",
  "patternType": "error-handler",
  "label": "Error Handler",
  "handlerFunc": "msg.payload = { error: msg.error.message, timestamp: new Date() };\nreturn msg;"
}
```

### transform - 변환 파이프라인

```
inject → transform1 → transform2 → ... → debug
```

```json
{
  "action": "pattern_build",
  "patternType": "transform",
  "label": "Data Pipeline",
  "transforms": [
    {
      "name": "Parse",
      "func": "msg.payload = JSON.parse(msg.payload);\nreturn msg;"
    },
    {
      "name": "Filter",
      "func": "msg.payload = msg.payload.filter(x => x.active);\nreturn msg;"
    },
    {
      "name": "Format",
      "func": "msg.payload = { count: msg.payload.length, data: msg.payload };\nreturn msg;"
    }
  ]
}
```

### parallel - 병렬 처리

```
inject → split → process → join → debug
```

```json
{
  "action": "pattern_build",
  "patternType": "parallel",
  "label": "Parallel Processor",
  "handlerFunc": "msg.payload = msg.payload * 2;\nreturn msg;"
}
```

---

## NodeFactory 노드 타입

`node_create` 또는 `node_types`로 사용 가능한 40+ 기본 노드 타입:

### Common (공통)

| 타입        | 설명               |
| ----------- | ------------------ |
| `inject`    | 메시지 주입/타이머 |
| `debug`     | 디버그 출력        |
| `complete`  | 노드 완료 감지     |
| `catch`     | 에러 캐치          |
| `status`    | 노드 상태 감지     |
| `link_in`   | 링크 입력          |
| `link_out`  | 링크 출력          |
| `link_call` | 링크 호출          |
| `comment`   | 주석               |
| `junction`  | 연결점             |

### Function (함수)

| 타입       | 설명             |
| ---------- | ---------------- |
| `function` | JavaScript 함수  |
| `change`   | 속성 설정/변경   |
| `switch`   | 조건 분기        |
| `range`    | 값 범위 변환     |
| `template` | Mustache 템플릿  |
| `delay`    | 지연/속도 제한   |
| `trigger`  | 트리거           |
| `exec`     | 시스템 명령 실행 |
| `rbe`      | 중복 제거        |

### Network (네트워크)

| 타입                              | 설명            |
| --------------------------------- | --------------- |
| `httpIn`                          | HTTP 엔드포인트 |
| `httpResponse`                    | HTTP 응답       |
| `httpRequest`                     | HTTP 클라이언트 |
| `websocketIn` / `websocketOut`    | WebSocket       |
| `tcpIn` / `tcpOut` / `tcpRequest` | TCP             |
| `udpIn` / `udpOut`                | UDP             |
| `mqttIn` / `mqttOut`              | MQTT            |

### Sequence (시퀀스)

| 타입    | 설명             |
| ------- | ---------------- |
| `split` | 배열/문자열 분할 |
| `join`  | 메시지 결합      |
| `sort`  | 정렬             |
| `batch` | 배치 처리        |

### Parser (파서)

| 타입   | 설명      |
| ------ | --------- |
| `json` | JSON 변환 |
| `csv`  | CSV 변환  |
| `html` | HTML 파싱 |
| `xml`  | XML 변환  |
| `yaml` | YAML 변환 |

### Storage (저장소)

| 타입     | 설명      |
| -------- | --------- |
| `file`   | 파일 쓰기 |
| `fileIn` | 파일 읽기 |
| `watch`  | 파일 감시 |

---

## 템플릿 (10개)

`templates_list`와 `template_apply`로 사용:

| ID                | 이름          | 설명                |
| ----------------- | ------------- | ------------------- |
| `http-api`        | HTTP REST API | 기본 HTTP 요청/응답 |
| `http-api-crud`   | CRUD API      | 완전한 CRUD 패턴    |
| `mqtt-processor`  | MQTT 처리기   | MQTT 메시지 처리    |
| `timer-task`      | 타이머 작업   | 주기적 자동화       |
| `webhook-handler` | 웹훅 핸들러   | 웹훅 분기 처리      |
| `error-handler`   | 에러 핸들러   | 에러 캐치/로깅      |
| `http-proxy`      | HTTP 프록시   | 외부 API 프록시     |
| `mqtt-to-http`    | MQTT→HTTP     | 프로토콜 브릿지     |
| `data-logger`     | 데이터 로거   | 파일 저장           |
| `rate-limiter`    | 속도 제한     | 메시지 제한         |

---

## 사용 워크플로우

### 패턴으로 빠른 생성

```
1. pattern_build(patternType: "http-api", baseUrl: "/api/data")
2. flows_deploy(flows: [...])  # 반환된 flows 배포
```

### 수동 플로우 구성

```
1. flow_create(label: "My Flow")           # 탭 생성
2. node_create(nodeType: "inject", ...)    # 노드 생성
3. node_create(nodeType: "function", ...)
4. node_create(nodeType: "debug", ...)
5. nodes_connect(sourceId, targetId)       # 연결
6. flows_deploy(flows: [...])              # 배포
```

### 노드 정보 조회

```
1. node_types()                            # 사용 가능 타입 목록
2. catalog_search(query: "http")           # 노드 검색
3. catalog_info(nodeType: "function")      # 상세 정보
```

---

## 에러 처리

- **401**: 인증 실패 - 토큰 확인
- **409**: 리비전 충돌 - flows_get으로 최신 rev 가져온 후 재시도
- **readOnly 모드**: 쓰기 작업 차단됨

---

## 관련 링크

- [Node-RED Admin API](https://nodered.org/docs/api/admin/)
- [Node-RED 노드 개발](https://nodered.org/docs/creating-nodes/)


##  핵심 키워드
### PUT과  PATCH의 차이

|  | PUT | PATCH |
| --- | --- | --- |
| 정의 | 특정 리소스를 전체적으로 교체하는 HTTP 메서드 | 일부 데이터를 사용하여 리소스를 부분적으로 수정하는 HTTP 메서드 |
| 목적 | 리소스를 전체적으로 교체 | 리소스를 부분적으로 수정 |
| 요청 본문 | 전체 리소스 데이터를 포함 | 변경할 부분만 포함 |
| 리소스 존재 여부 | 리소스가 없으면 새로 생성 | 리소스가 없으면 수정하지 않음 |
| 안정성 | 기존 데이터를 덮어쓸 수 있음 | 덮어쓰지 않고 일부만 변경 |
| 멱등성 | 멱등성 보장 O | 보장 X |

### **멱등성**
    - “동일한 요청을 여러 번 보내도 결과가 항상 같다”
    - 같은 요청을 반복해서 보내도 리소스의 상태가 변경되지 않거나 얻은 결과가 동일하게 유지되는 특성
    - PATCH가 멱등성을 보장하지 않는 이유
        - ex) PATCH 요청을 통해 나이가 1씩 증가하는 동작을 구현한 경우 같은 요청을 여러 번 보내면 결과가 달라지기 때문에 멱등성을 가지지 않게 된다.
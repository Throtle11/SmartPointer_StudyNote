# unique_ptr

unique_ptr은 어떤 객체를 '딱 한 곳에서만' 소유하도록 만드는 스마트포인터이다.  
스코프를 벗어나면 자동으로 delete되어 누수를 줄인다. (RAII적용)

## 왜 중요한가
new로 만든 객체는 delete를 직접 해야 한다.  
하지만 중간 return이나 예외가 나면 delete를 빼먹기 쉽다.  
unique_ptr을 쓰면 스코프 종료 시 자동 해제되어 이런 실수를 줄일 수 있다.

## 핵심 정리
- unique_ptr은 단독 소유(소유자 1명)이다.
- 그래서 복사는 불가능하다. (중복 해제 위험)
- 대신 이동(move)은 가능하다. (소유권을 넘기는 것)
- 생성은 보통 std::make_unique<T>()를 사용한다.

## 예제 

#include <iostream>
#include <memory>
using namespace std;

class A {
public:
    A()  { cout << "생성" << endl; }
    ~A() { cout << "소멸" << endl; }
};

int main() {
    auto p = make_unique<A>(); // A를 만들고 unique_ptr로 소유
    cout << "작업중" << endl;
} // 여기서 p가 사라지며 자동 소멸

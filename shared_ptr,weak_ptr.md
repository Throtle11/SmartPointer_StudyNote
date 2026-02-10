# shared_ptr / weak_ptr

- shared_ptr은 하나의 객체를 여러 곳에서 '함께 소유'하는 스마트포인터이다. 
- weak_ptr은 소유권 없이 shared_ptr을 '참조만' 하는 포인터로 순환 참조 문제를 막는 데 사용한다.

## 왜 중요한가
객체를 여러 곳에서 같이 써야 할 때 shared_ptr을 쓰면 마지막 소유자가 사라질 때 자동으로 해제되어 편리하다.  
하지만 shared_ptr끼리 서로를 소유하면(서로를 shared_ptr로 들고 있으면) 참조 카운트가 0이 되지 않아 객체가 해제되지 않을 수 있다.  
이런 순환 참조 문제를 끊기 위해 weak_ptr을 사용한다.

## 핵심 정리
- shared_ptr을 복사하면 참조 카운트가 증가한다.
- 마지막 shared_ptr이 사라질 때 객체가 소멸한다.
- weak_ptr은 참조 카운트를 증가시키지 않는다(소유하지 않는다).
- weak_ptr은 사용하기 전에 lock()으로 shared_ptr을 얻어서 확인 후 사용한다.
- expired()가 true면 이미 객체가 소멸된 상태다.

## 예제 
-----------------------shared_ptr---------------------------------
#include <iostream>
#include <memory>
using namespace std;

class A {
public:
    A()  { cout << "A 생성" << endl; }
    ~A() { cout << "A 소멸" << endl; }
};

int main() {
    auto p1 = make_shared<A>();
    auto p2 = p1; // 공유 소유(참조 카운트 증가)

    cout << "끝" << endl;
     return 0;
} // p1, p2가 모두 사라진 뒤 A 소멸

------------------------weak_ptr----------------------------------
#include <iostream>
#include <memory>
using namespace std;

class A {
public:
    A()  { cout << "A 생성" << endl; }
    ~A() { cout << "A 소멸" << endl; }
};

int main() {
    weak_ptr<A> wp; // 소유권 없음(카운트 증가 X)

    {
        auto sp = make_shared<A>(); // 여기서 A 생성
        wp = sp; // wp는 sp를 "참조만" 함

        cout << "블록 안: ";
        if (auto locked = wp.lock()) { // 살아있으면 shared_ptr로 잠깐 얻음
            cout << "lock 성공" << endl;
        } else {
            cout << "lock 실패" << endl;
        }
    } // 여기서 sp가 사라짐 >>>> A 소멸(weak_ptr만 남았으므로)

    cout << "블록 밖: ";
    if (wp.expired()) {
        cout << "이미 소멸됨" << endl;
    } else {
        cout << "아직 살아있음" << endl;
    }

    cout << "블록 밖 lock: ";
    if (auto locked = wp.lock()) {
        cout << "lock 성공" << endl;
    } else {
        cout << "lock 실패" << endl;
    }
    return 0;
}

# custom deleter 

custom deleter는 스마트포인터가 객체를 해제할 때 delete 대신 '내가 지정한 해제 함수'를 호출하도록 만드는 기능이다.

## 왜 중요한가
모든 자원이 delete로 해제되는 것은 아니다.  
예를 들어 파일은 fclose(), Windows 핸들은 CloseHandle() 같은 전용 해제 함수가 필요하다.  
이때 custom deleter 를 쓰면 '해제 코드를 소멸자에 맡기는 RAII'를 스마트포인터로 그대로 적용할 수 있다.

## 언제 쓰나(대표 상황)
- FILE* : fclose()로 닫아야 한다.
- Windows HANDLE : CloseHandle()로 닫아야 한다.
- 라이브러리가 만든 자원 : Library_Free() 같은 전용 해제 함수가 있다.

## 핵심 정리
- unique_ptr은 두 번째 템플릿 인자로 'deleter 타입'을 가질 수 있다.
- 보통 함수 포인터 또는 람다(함수 객체)를 deleter로 쓴다.
- 포인터가 nullptr이면 해제 함수는 호출하지 않는 형태로 작성하는 게 안전하다.

## 예제 
#include <iostream>
#include <memory>
using namespace std;

// delete로 지우면 안 되는 자원이라고 가정 (예: OS 핸들/라이브러리 핸들)
struct Resource {
    int id;
};

// 전용 해제 함수(이걸 반드시 써야 함)
void Release(Resource* r) {
    cout << "Release 호출 (id=" << r->id << ")" << endl;
    delete r; 
}

int main() {
    // custom deleter: unique_ptr이 사라질 때 Release를 호출하게 함
    auto deleter = [](Resource* r) { Release(r); };

    unique_ptr<Resource, decltype(deleter)> p(new Resource{1}, deleter);

    cout << "작업중" << endl;

    return 0; // 여기서 p 소멸 >>> deleter 실행 >>> Release 호출
}

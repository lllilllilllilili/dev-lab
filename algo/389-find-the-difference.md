389. Find the Difference (Java)
     문제 요약
     문자열 t는 문자열 s를 랜덤하게 섞은 뒤, 문자 하나를 추가해서 만든 문자열이다.
     추가된 문자 1개를 찾아서 반환한다.
     핵심 아이디어
     XOR(^) 연산을 이용하면 같은 문자는 서로 상쇄된다.

성질:
a ^ a = 0
a ^ 0 = a
XOR은 순서와 상관없이 결과가 같다 (교환/결합 가능)
즉, s와 t의 모든 문자를 XOR 하면
짝이 맞는 문자들은 전부 0이 되고, 추가된 문자만 남는다.

코드
```aiignore
lass Solution {
public char findTheDifference(String s, String t) {
int x = 0;

        for (int i = 0; i < s.length(); i++) {
            x ^= s.charAt(i);
        }

        for (int i = 0; i < t.length(); i++) {
            x ^= t.charAt(i);
        }

        return (char) x;
    }
}
```
동작 예시
s = "abcd"
t = "abcde"
계산:
a ^ b ^ c ^ d ^ a ^ b ^ c ^ d ^ e = e

결과: 'e'

복잡도
시간복잡도: O(n) (s, t를 한 번씩 순회)
공간복잡도: O(1) (추가 변수 1개만 사용)
한 줄 요약
정렬/해시 없이, XOR 누적으로 추가된 문자 1개를 상수 공간에서 찾는 풀이.
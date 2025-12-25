```java
class Solution {
public String longestCommonPrefix(String[] strs) {
if (strs == null || strs.length == 0) {
return "";
}

        // 첫 번째 문자열을 기준으로 사용
        String base = strs[0];
        
        // 첫 번째 문자열의 각 문자를 순회
        for (int i = 0; i < base.length(); i++) {
            char c = base.charAt(i);
            
            // 모든 문자열의 i번째 문자 검사
            for (int j = 1; j < strs.length; j++) {
                // i가 문자열 길이를 벗어나거나 문자가 일치하지 않으면 종료
                if (i >= strs[j].length() || strs[j].charAt(i) != c) {
                    return base.substring(0, i);
                }
            }
        }
        
        // 모든 문자가 일치하면 첫 번째 문자열 전체 반환
        return base;
    }
}
```
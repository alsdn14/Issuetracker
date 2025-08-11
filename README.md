이슈 트래킹 자동화 코드입니다  
  
Cursor와 ChatGPT 기반의 Vibe Coding 방식으로 개발했으며, 실무에서 활용한 자동화 프로세스를 구현한 예시 코드 입니다.  
  
보안을 위해 GitLab Token, 프로젝트 ID, Google Sheets 등은 예시값으로 대체되어 있습니다.  

# GitLab–Google Sheets Issue Tracker

GitLab 프로젝트 이슈 현황을 Google Sheets와 자동 동기화하는 Python 스크립트입니다.  
마일스톤 변경, 상태 업데이트, 신규 이슈 추가를 자동화해 QA 및 프로젝트 관리 효율성을 높입니다.

---

## 기능
- **GitLab API 연동**: 특정 마일스톤의 모든 이슈 조회
- **Google Sheets API 연동**: 시트에 자동 기록 및 변경 사항 업데이트
- **변경 사항만 업데이트**하여 API 호출 최적화
- 마일스톤 변경 감지 시 자동 표시
- 특정 레이블(빌드미포함, 재연필요) 자동 필터링
- 완료 조건(‘완료대기+QA’ 또는 closed 상태) 자동 판별

---

## 실행 흐름
1. Google Sheets 인증 (서비스 계정 JSON 키 사용)
2. 기존 시트 데이터 로드
3. GitLab API로 이슈 목록 조회
4. 기존 데이터와 비교 → 변경된 이슈만 업데이트
5. 신규 이슈 자동 추가
6. 마일스톤 변경 감지 및 표시
7. 결과 출력


import time
import gitlab
import gspread
from oauth2client.service_account import ServiceAccountCredentials

# GitLab 연동 설정 (보안 정보는 예시값으로 대체)
GITLAB_URL = "https://your.gitlab.instance"
ACCESS_TOKEN = "your_gitlab_token"
PROJECT_ID = 123  # 예시 프로젝트 ID

# Google Sheets 연동 설정 ( 가데이터 )
SHEET_KEY = "your_google_sheet_key"
JSON_KEYFILE = "your_keyfile_path.json"

# Google Sheets 인증 ( 가데이터 )
scope = ["https://spreadsheets.google.com/feeds", "https://www.googleapis.com/auth/drive"]
creds = ServiceAccountCredentials.from_json_keyfile_name(JSON_KEYFILE, scope)
client = gspread.authorize(creds)
sheet = client.open_by_key(SHEET_KEY).worksheet("Row") # 시트 이름 Row 로 임의 설정

# 기존 데이터 읽기 (시트에서 가져오기 때문에 이중 리스트 형태로 저장됨)
data = sheet.get_all_values()
if not data:
    header = ["이슈 번호", "상태", "라벨 정보", "마일스톤", "담당자", "제목"] # 예시 라벨
    sheet.update("A1:F1", [header])
    data = [header]

header, rows = data[0], data[1:]
existing_data = {row[0]: row for row in rows if len(row) >= 6}
existing_milestones = {row[0]: row[3] for row in rows if len(row) >= 4}

# GitLab 연결
gl = gitlab.Gitlab(GITLAB_URL, private_token=ACCESS_TOKEN)
gl.auth()
project = gl.projects.get(PROJECT_ID)

# 이슈 가져오기
MILESTONE_TITLE = "example25.8" #마일스톤 입력 예시 데이터
issues = project.issues.list(milestone=MILESTONE_TITLE, all=True)

update_data = []
new_rows = []
current_issue_ids = set()

for issue in issues:
    if "빌드미포함" in issue.labels or "재연필요" in issue.labels:
        continue
    issue_id = str(issue.iid)
    labels = ", ".join(issue.labels)
    assignee = issue.assignees[0]["name"] if issue.assignees else ""
    milestone = issue.milestone["title"] if issue.milestone else ""
    title = issue.title
    current_issue_ids.add(issue_id)
    status = "Pass" if ("완료대기" in issue.labels and "QA" in issue.labels) or issue.state == "closed" else "N/T"

    new_row = [issue_id, status, labels, milestone, assignee, title]
    if issue_id in existing_data:
        existing_row = existing_data[issue_id]
        if existing_row != new_row:
            row_index = rows.index(existing_row) + 2
            range_str = f"A{row_index}:F{row_index}"
            update_data.append((range_str, [new_row]))
    else:
        new_rows.append(new_row)

# 마일스톤 변경 될 때
for issue_id, old_milestone in existing_milestones.items():
    if old_milestone == MILESTONE_TITLE and issue_id not in current_issue_ids:
        row_index = rows.index(existing_data[issue_id]) + 2
        update_data.append((f"B{row_index}", [["마일스톤변경"]]))

# 시트 업데이트
if update_data:
    batch_updates = [{"range": r, "values": v} for r, v in update_data]
    for i in range(0, len(batch_updates), 10):
        sheet.batch_update(batch_updates[i:i + 10], value_input_option="USER_ENTERED")
        time.sleep(2)

if new_rows:
    time.sleep(2)
    sheet.append_rows(new_rows, value_input_option="USER_ENTERED")

print("이슈가 업데이트되었습니다.")

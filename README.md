# cd-test-1
코디세이 온보딩 수업 1
#!/usr/bin/env python3
# attend.py
# 스터디 출석부 프로그램
# - 외부 라이브러리 사용 없음
# - 날짜와 이름을 저장
# - 저장된 출석 목록 조회 가능

import json
import os
import sys
from datetime import datetime

DATA_FILE = os.path.join(os.path.dirname(__file__), "attendance.json")


def load_records():
    """저장된 출석 기록 불러오기"""
    if not os.path.exists(DATA_FILE):
        return []

    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            data = json.load(f)
            if isinstance(data, list):
                return data
            return []
    except (json.JSONDecodeError, OSError):
        return []


def save_records(records):
    """출석 기록 저장하기"""
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(records, f, ensure_ascii=False, indent=2)


def validate_date(date_text):
    """날짜 형식 확인: YYYY-MM-DD"""
    try:
        datetime.strptime(date_text, "%Y-%m-%d")
        return True
    except ValueError:
        return False


def add_record(date_text, name):
    """출석 기록 추가"""
    date_text = date_text.strip()
    name = name.strip()

    if not date_text or not name:
        print("날짜와 이름을 모두 입력해야 합니다.")
        return

    if not validate_date(date_text):
        print("날짜 형식이 올바르지 않습니다. 예: 2026-11-02")
        return

    records = load_records()

    record = {
        "date": date_text,
        "name": name,
        "created_at": datetime.now().isoformat(timespec="seconds")
    }
    records.append(record)
    save_records(records)

    print(f"출석 저장 완료: {date_text} / {name}")


def list_records():
    """출석 목록 출력"""
    records = load_records()

    if not records:
        print("저장된 출석 기록이 없습니다.")
        return

    # 날짜, 이름 순으로 정렬
    records = sorted(records, key=lambda x: (x.get("date", ""), x.get("name", ""), x.get("created_at", "")))

    print("\n[출석 목록]")
    print("-" * 50)
    print(f"{'No':<4} {'날짜':<12} {'이름':<20} {'기록시간'}")
    print("-" * 50)

    for i, r in enumerate(records, start=1):
        date_text = r.get("date", "")
        name = r.get("name", "")
        created_at = r.get("created_at", "")
        print(f"{i:<4} {date_text:<12} {name:<20} {created_at}")

    print("-" * 50)
    print(f"총 {len(records)}건")


def help_text():
    """사용법 안내"""
    print("사용법:")
    print("  python attend.py add YYYY-MM-DD 이름")
    print("  python attend.py list")
    print()
    print("예시:")
    print("  python attend.py add 2026-11-02 홍길동")
    print("  python attend.py list")


def interactive_mode():
    """명령이 없을 때 간단한 대화형 모드"""
    while True:
        print("\n[스터디 출석부]")
        print("1. 출석 추가")
        print("2. 출석 목록 보기")
        print("3. 종료")

        choice = input("선택하세요: ").strip()

        if choice == "1":
            date_text = input("날짜 (YYYY-MM-DD): ").strip()
            name = input("이름: ").strip()
            add_record(date_text, name)
        elif choice == "2":
            list_records()
        elif choice == "3":
            print("종료합니다.")
            break
        else:
            print("1, 2, 3 중에서 선택해 주세요.")


def main():
    args = sys.argv[1:]

    if not args:
        interactive_mode()
        return

    command = args[0].lower()

    if command == "add":
        if len(args) < 3:
            print("출석 추가에는 날짜와 이름이 필요합니다.")
            help_text()
            return

        date_text = args[1]
        name = " ".join(args[2:])
        add_record(date_text, name)

    elif command == "list":
        list_records()

    elif command in ("help", "-h", "--help"):
        help_text()

    else:
        print(f"알 수 없는 명령: {command}")
        help_text()


if __name__ == "__main__":
    main()
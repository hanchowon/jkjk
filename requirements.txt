from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Any, Dict

app = FastAPI(title="Linear Search Step Visualizer API")

# Google Apps Script 및 외부 웹 앱과의 통신을 위한 CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 모든 오리진 허용 (필요시 특정 도메인으로 제한 가능)
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 요청 데이터 구조 정의
class SearchRequest(BaseModel):
    array: List[int]  # 검색 대상 정수 배열
    target: int      # 찾고자 하는 목표 값

@app.get("/")
def read_root():
    """서버 상태 확인용 헬스체크 엔드포인트"""
    return {"status": "online", "message": "Linear Search Server is running"}

@app.post("/search/linear")
def linear_search(data: SearchRequest) -> Dict[str, Any]:
    """
    선형 검색(Linear Search)을 수행하고, 
    각 단계별(Step-by-step) 진행 상황과 복잡도 정보를 반환합니다.
    """
    array = data.array
    target = data.target
    steps = []
    found = False
    found_index = -1

    # 선형 검색 수행: 배열의 처음부터 끝까지 순차적으로 비교
    for i in range(len(array)):
        current_val = array[i]
        is_match = (current_val == target)
        
        # 현재 단계의 상세 정보 기록
        step_info = {
            "step": i + 1,            # 1부터 시작하는 단계 번호
            "current_index": i,       # 현재 비교 중인 배열 인덱스
            "current_value": current_val, # 현재 비교 중인 값
            "is_match": is_match      # 일치 여부
        }
        steps.append(step_info)

        if is_match:
            found = True
            found_index = i
            break  # 목표 값을 찾았으므로 검색 종료

    # 알고리즘 시간 및 공간 복잡도 정의
    complexity = {
        "time_best": "O(1)",          # 최선의 경우: 첫번째 요소를 찾을 때
        "time_worst": "O(N)",         # 최악의 경우: 마지막 요소이거나 없을 때
        "time_average": "O(N)",       # 평균적인 경우
        "space": "O(1)"               # 제자리 검색으로 추가 메모리 최소화
    }

    return {
        "success": True,
        "target": target,
        "found": found,
        "found_index": found_index,
        "total_steps": len(steps),
        "steps": steps,
        "complexity": complexity
    }

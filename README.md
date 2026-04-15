from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="Win Probability Prediction API")

# -----------------------------
# Request Schema
# -----------------------------
class WinPredictionRequest(BaseModel):
    current_score: int
    wickets: int
    overs: float
    target: int


# -----------------------------
# Response Schema
# -----------------------------
class WinPredictionResponse(BaseModel):
    runs_needed: int
    balls_remaining: int
    required_run_rate: float
    prediction: str


# -----------------------------
# Endpoint
# -----------------------------
@app.post("/predict/win", response_model=WinPredictionResponse)
def predict_win(data: WinPredictionRequest):

    # Total balls in ODI/T20 style match (assumption: 20 overs format = 120 balls)
    total_balls = 120
    balls_completed = int(data.overs * 6)
    balls_remaining = total_balls - balls_completed

    runs_needed = data.target - data.current_score

    # Avoid division errors
    if balls_remaining > 0:
        required_run_rate = (runs_needed / balls_remaining) * 6
    else:
        required_run_rate = 0

    # Simple rule-based prediction logic
    if runs_needed <= 0:
        prediction = "Already Won"
    elif balls_remaining <= 0:
        prediction = "Lost Chance"
    elif required_run_rate <= 7:
        prediction = "High Chance"
    elif required_run_rate <= 10:
        prediction = "Medium Chance"
    else:
        prediction = "Low Chance"

    return WinPredictionResponse(
        runs_needed=runs_needed,
        balls_remaining=balls_remaining,
        required_run_rate=round(required_run_rate, 2),
        prediction=prediction
    )

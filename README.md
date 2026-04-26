# === ENUM VALUES ===
UP = "UP"
DOWN = "DOWN"

FIRST = "FIRST"
SECOND = "SECOND"

ABOVE = "ABOVE"
UNDER = "UNDER"

CONTINUATION = "CONTINUATION"
REVERSAL = "REVERSAL"

OVERBOUGHT = "OVERBOUGHT"
OVERSOLD = "OVERSOLD"
NONE = "NONE"


# === STATE OBJECT ===
class MarketState:
    def __init__(self, daily_trend, h4_trend, zone,
                 trendline, pattern,
                 break_line=False,
                 daily_condition=NONE):
        self.daily_trend = daily_trend
        self.h4_trend = h4_trend
        self.zone = zone
        self.trendline = trendline
        self.pattern = pattern
        self.break_line = break_line
        self.daily_condition = daily_condition


# === HELPER ===
def valid_zone(state):
    return (
        state.zone == FIRST or
        (state.zone == SECOND and state.break_line)
    )


# === STRATEGY ENGINE ===
def analyze(state: MarketState):

    # 1. Bearish continuation
    if (
        state.daily_trend == DOWN and
        state.h4_trend == DOWN and
        state.trendline == UNDER and
        state.pattern == CONTINUATION and
        valid_zone(state) and
        state.daily_condition != OVERSOLD
    ):
        return {
            "signal": "SELL",
            "type": "Bearish Continuation"
        }

    # 2. Deep pullback continuation
    if (
        state.daily_trend == UP and
        state.h4_trend == UP and
        state.trendline == ABOVE and
        state.pattern == CONTINUATION and
        valid_zone(state) and
        state.daily_condition != OVERBOUGHT
    ):
        return {
            "signal": "BUY",
            "type": "Deep Pullback Continuation"
        }

    # 3. Counter reversal (advanced)
    if (
        state.daily_trend == UP and
        state.h4_trend == DOWN and
        state.trendline == UNDER and
        state.pattern == REVERSAL and
        valid_zone(state)
    ):
        return {
            "signal": "SELL",
            "type": "Counter Reversal"
        }

    # 4. Early reversal (top/bottom catch)
    if (
        state.daily_trend == DOWN and
        state.h4_trend == UP and
        state.trendline == ABOVE and
        state.pattern == REVERSAL and
        valid_zone(state)
    ):
        return {
            "signal": "BUY",
            "type": "Early Reversal"
        }

    return {
        "signal": "NO TRADE",
        "type": None
    }


# === TEST EXAMPLE ===
if __name__ == "__main__":

    state = MarketState(
        daily_trend=DOWN,
        h4_trend=DOWN,
        zone=FIRST,
        trendline=UNDER,
        pattern=CONTINUATION,
        break_line=False,
        daily_condition=NONE
    )

    result = analyze(state)

    print("Signal:", result["signal"])
    print("Setup :", result["type"])

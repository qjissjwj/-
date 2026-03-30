import random
import pandas as pd
from psychopy import visual, event, core
from config import (SET_SIZES, TRIALS_TARGET_ON, TRIALS_TARGET_OFF,
                    CONDITIONS, SCREEN_SIZE, BG_COLOR)
from trial import run_single_trial


def show_message(win, text, wait_key=True):
    msg = visual.TextStim(
        win,
        text       = text,
        height     = 35,
        color      = (1, 1, 1),
        colorSpace = 'rgb',
        wrapWidth  = 1200,
    )
    msg.draw()
    win.flip()
    if wait_key:
        event.waitKeys()


def build_trial_list():
    """
    순서:
    set_size=4:
        조건1 15회 → 조건2 15회 → 조건3 15회 → 조건4 15회
    set_size=8:
        조건1 15회 → 조건2 15회 → 조건3 15회 → 조건4 15회
    set_size=12:
        조건1 15회 → 조건2 15회 → 조건3 15회 → 조건4 15회
    총 180회, 각 조건 15회 안에서만 무선화
    """
    all_trials = []

    for set_size in SET_SIZES:              # 4 → 8 → 12
        for (target_f, dist_f) in CONDITIONS:   # 조건1 → 조건2 → 조건3 → 조건4
            block = []
            for _ in range(TRIALS_TARGET_ON):   # 표적있음 8회
                block.append(dict(
                    target_font     = target_f,
                    distractor_font = dist_f,
                    set_size        = set_size,
                    target_present  = True
                ))
            for _ in range(TRIALS_TARGET_OFF):  # 표적없음 7회
                block.append(dict(
                    target_font     = target_f,
                    distractor_font = dist_f,
                    set_size        = set_size,
                    target_present  = False
                ))
            random.shuffle(block)           # 15회 안에서만 무선화
            all_trials.extend(block)

    return all_trials
    # 결과: [set4-조건1×15, set4-조건2×15, set4-조건3×15, set4-조건4×15,
    #        set8-조건1×15, set8-조건2×15, set8-조건3×15, set8-조건4×15,
    #        set12-조건1×15, set12-조건2×15, set12-조건3×15, set12-조건4×15]


def run_experiment():
    win = visual.Window(
        size       = SCREEN_SIZE,
        fullscr    = True,
        color      = BG_COLOR,
        colorSpace = 'rgb',
        units      = 'pix',
    )

    fixation = visual.TextStim(
        win,
        text       = '+',
        height     = 30,
        color      = (1, 1, 1),
        colorSpace = 'rgb',
    )

    # 1. 전체 안내
    show_message(win,
        "[ 시각 탐색 실험 ]\n\n"
        "여러 글자 중에서 글꼴이 다른 글자(표적)를 찾으세요.\n\n"
        "표적이 있으면   →   Z 키\n"
        "표적이 없으면   →   M 키\n\n"
        "최대한 빠르고 정확하게 반응해 주세요.\n\n"
        "아무 키나 눌러 계속하세요."
    )

    # 2. 연습 안내
    show_message(win,
        "[ 연습 시작 ]\n\n"
        "본 실험 전 연습을 진행합니다.\n"
        "총 10회 진행됩니다.\n\n"
        "아무 키나 눌러 시작하세요."
    )

    # 3. 연습 10회
    practice_trials = build_trial_list()[:10]
    for trial in practice_trials:
        result = run_single_trial(win=win, fixation=fixation, **trial)
        if result.get('key') == 'escape':
            win.close()
            return

    # 4. 본실험 안내
    show_message(win,
        "[ 연습 완료 ]\n\n"
        "수고하셨습니다! 이제 본 실험을 시작합니다.\n\n"
        "표적이 있으면   →   Z 키\n"
        "표적이 없으면   →   M 키\n\n"
        "아무 키나 눌러 본 실험을 시작하세요."
    )

    # 5. 본 실험 180회
    results = []
    trials  = build_trial_list()

    for i, trial in enumerate(trials):
        result = run_single_trial(win=win, fixation=fixation, **trial)
        result.update(trial)
        result['trial_n'] = i + 1
        results.append(result)

        if result.get('key') == 'escape':
            break

    # 6. 종료 안내
    show_message(win,
        "[ 실험 완료 ]\n\n"
        "수고하셨습니다!\n"
        "잠시 후 화면이 종료됩니다.",
        wait_key=False
    )
    core.wait(2.0)
    win.close()

    # 7. 결과 저장
    df = pd.DataFrame(results)
    df.to_csv("data/results.csv", index=False, encoding='utf-8-sig')
    print(f"실험 완료! 총 {len(results)}개 시행 → data/results.csv 저장됨")


if __name__ == "__main__":
    run_experiment()

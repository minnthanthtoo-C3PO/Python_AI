# Python_AI
import unicodedata

# Load dictionary (Myanmar words in Unicode)
def load_dictionary(path: str) -> set[str]:
    # robust file load + cleanup
    try:
        with open(path, "r", encoding="utf-8") as f:
            words = []
            for line in f:
                w = unicodedata.normalize("NFC", line.strip())
                if not w:
                    continue
                w = re.sub(r"\s*\(.*?\)\s*$", "", w)
                if w:
                    words.append(w)
        return set(words)
    except FileNotFoundError:
        print(f"Dictionary file not found: {path}")
        return set()
# print(len(burmese_dict), "words loaded")
# print(list(burmese_dict)[:10])

# word_to_be_checked = input("Enter a word you want to check: ")

def check_word(word: str, dictionary: set[str]) -> bool:
    w = unicodedata.normalize("NFC", word.strip())
    return w in dictionary

def levenshtein(a: str, b: str) -> int:
    if a == b: return 0
    la, lb = len(a), len(b)
    if la == 0: return lb
    if lb == 0: return la
    # ensure a is shorter for minor speed win
    if la > lb: a, b, la, lb = b, a, lb, la
    prev = list(range(la + 1))
    for j in range(1, lb + 1):
        cur = [j] + [0]*la
        bj = b[j-1]
        for i in range(1, la + 1):
            cost = 0 if a[i-1] == bj else 1
            cur[i] = min(prev[i] + 1,      # deletion
                         cur[i-1] + 1,     # insertion
                         prev[i-1] + cost) # substitution
        prev = cur
    return prev[la]

def suggest(word: str, dictionary: set[str], k: int = 5, max_ed: int = 2) -> list[str]:
    w = unicodedata.normalize("NFC", word.strip())
    if not w or not dictionary:
        return []
    # length prefilter to keep it fast
    cands = (dw for dw in dictionary if abs(len(dw) - len(w)) <= max_ed)
    scored = []
    for dw in cands:
        d = levenshtein(w, dw)
        if d <= max_ed:
            scored.append((d, dw))
    scored.sort(key=lambda t: (t[0], t[1]))  # distance ASC, then alphabetical
    return [dw for _, dw in scored[:k]]

def explain(word: str) -> str:
    if check_word(word, burmese_dict):
        return f"'{word}' ✅ စာလုံးပေါင်းမှန်ပါတယ်ခင်ဗျာ။"
    else:
        alts = suggest(word, burmese_dict, k=5, max_ed=2)
        if alts:
            return f"'{word}' ❌ မှားပါတယ်ခင်ဗျာ။ အနီးစပ်ဆုံးအကြံပြုချက်များ ➜ {', '.join(alts)}"
        return f"'{word}' ❌ မှားပါတယ်ခင်ဗျာ။ (မည်သည့်အကြံပြုချက်မျှမတွေ့ပါ)"

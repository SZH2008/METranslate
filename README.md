[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SZH2008/METranslate/blob/main/METranslate.ipynb)

# METranslate: Can families like ours trust a free translator?

**Track:** Languages (Veritas AI International High School Hackathon 2026)

**Team:** Abdullah Shoaib, Sultan Ul-Arfeen Malik and Syed Zain-ul-Abideen Hussain, from Pakistan International School Jeddah (English Section)

## Demo video
https://github.com/user-attachments/assets/958ffc4c-0929-4a57-ab7a-272167ff3f42

**In short:** we tested a free translator and it did okay in Arabic (53.5 chrF) but worse in Urdu (47.2). When we gave it school notices, 5 out of 20 came out wrong enough to confuse a parent. The worst one turned a nut allergy into a mint allergy! Our tool catches some of these mistakes, but you still need a real person to check.

---

## Why we picked this

All three of us are from Pakistan, and we moved to Saudi Arabia when we were kids. So our Arabic isn't that good, and our Urdu isn't perfect either. English is kind of our best language now. There are a lot of families like ours here from Pakistan, India and Bangladesh, and a lot of the parents read Urdu way better than English or Arabic. But schools and offices still send everything in English or Arabic. If a translation gets a date, a fee or an allergy wrong, that could actually be a big deal.

Everyone knows free translators are good at big languages. We wanted to see how good one really is at Urdu compared to Arabic, and if a school could tell when a translation isn't safe to send home.

## What we made

We made METranslate. It's a simple tool a school office could use before sending notices to parents who speak Urdu or Arabic.

- **What you type in:** an English notice, then you pick Arabic or Urdu.
- **What you get:** the translation, the translation turned back into English (so you can see if the meaning stayed the same), and a result that says **OK** or **CHECK WITH A HUMAN**, plus why (a number changed, it's too long or repeating, or the meaning might be lost).

So the office only has to ask someone who speaks the language to check the ones that get flagged. We also checked how much worse the model is at Urdu than Arabic.

## What we added ourselves

We started with suggested project 4.4.2 (Translation quality test), and then we added these:

1. **Our own users and languages:** immigrant families in the Middle East. That's why we picked Arabic and Urdu, the two languages we grew up between.
2. **Checking numbers:** we looked at how often dates, times and amounts get messed up, and tried two fixes (beam search, and putting the right number back automatically).
3. **A warning that works without an answer key:** we translate the result back into English and compare it to the original. Then we tested if this warning actually finds the bad ones.
4. **A "too long or repeating" check:** we added this because the model got stuck in a loop on one of our notices.
5. **Real school notices and checking by people:** we wrote 10 school notices ourselves and our Urdu and Arabic readers checked them. Our Urdu speaker also rated 20 FLORES+ translations from 1 to 5.

## How it works

**Data:** we picked 100 random English sentences (seed 42) from the FLORES+ devtest set. Each sentence already comes with a professional Arabic (`arb_Arab`) and Urdu (`urd_Arab`) translation, so we used those as the answer key. 27 of the sentences have numbers. We also wrote 10 school notices ourselves, so the model had never seen them.

**Model:** `facebook/nllb-200-distilled-600M` (NLLB-200). It's a free model from Meta that can translate 200 languages. We didn't train it, we just tested it.

**Scoring:** we used chrF from the sacrebleu library. Basically it compares the model's translation to the professional one letter by letter and gives a score out of 100. For the baseline we just copied the English without translating, since any real translator should beat that easily. For the warning, we scored the back-translation against the original English, then used Spearman correlation to see if the warning agrees with the real chrF scores.

## Our results

Everything here is tested on the 100 FLORES+ sentences and our 10 notices. We didn't train anything on them.

| What we measured | Arabic | Urdu |
| --- | --- | --- |
| Baseline chrF (just copying the English) | 1.1 | 0.8 |
| NLLB chrF (normal) | 53.5 | 47.2 |
| NLLB chrF with beam search (5) | 54.6 | 47.4 |
| Sentences with numbers where all numbers were right | 81% | 89% |
| How much the warning agrees with real quality (Spearman) | 0.34 (p = 0.0005) | 0.21 (p = 0.034) |
| Worst 25% found by the warning (guessing gets about 6/25) | 9/25 | 8/25 |
| School notices a parent would understand (checked by us) | 6/10 | 9/10 |

- **The gap:** NLLB is way better than the baseline, but Urdu is still 6.3 points behind Arabic.
- **Our fixes:** beam search helped a tiny bit (+1.1 for Arabic, +0.2 for Urdu). Our number repair didn't change anything, because most of the number mistakes were numbers that went missing or got written as words, and our repair can't fix those.
- **The warning:** it finds more bad translations than just guessing, and it's statistically significant, but honestly it's pretty weak. It's more like a hint than something you can fully trust.
- **Our own ratings:** our Urdu speaker gave the 20 translations **4.65 / 5** on average, but chrF only gave the same ones 49.0 / 100. Our ratings and chrF barely agreed (Spearman 0.11, p = 0.64), but our ratings and the warning agreed more (0.44, p = 0.055, so almost significant). We think chrF is too strict on Urdu because there's more than one right way to spell and say things. We even found a spelling mistake in the professional answer key (دینا instead of دنیا)!
- **School notices:** 5 out of 20 translations would confuse a parent. Our checks caught 3 of them with no false alarms, but missed 2.

## Where it messes up (and why)

| English | What the model wrote (meaning) | What went wrong | Did we catch it? |
| --- | --- | --- | --- |
| Children with **nut** allergies must not eat food from other students. | Arabic: حساسية النعناع ("**mint** allergy") | Really dangerous word swap. The back-translation also said "mint", so our meaning check thought it was fine | No |
| **Pick-up** time on Wednesday is 12:30 pm. | Arabic: وقت الوصول ("**arrival** time") | A parent might come at the wrong time. The numbers and length were fine so nothing flagged it | No |
| The school bus will leave at 7:15 am instead of 7:45 am. | Arabic: "at seven and five and five and five…" | The model got stuck repeating itself and both times were lost | Yes (number check) |
| Your child **must** bring a packed lunch and a water bottle. | Arabic: "packed" was missing and our reader found it unclear | Part of the instruction got lost | Yes (meaning check) |
| Pick-up time on Wednesday is 12:30 pm. | Urdu: اٹھاؤ ("lifting"), back-translation: "time to pick up the teacher" | It translated "pick-up" word for word | Yes (meaning check) |
| …falling asleep a short time later (10–60 minutes). | Both: "(1060 minutes)" | The dash disappeared, so "10 to 60" turned into one thousand and sixty | Yes (number check) |

**Why we think this happens:** the model is fine with normal words, but it gets confused by words that mean more than one thing (like "pick-up") or words it mixes up (nut and mint). It also sometimes loses the dash or comma in the middle of numbers. Back-translation can't catch a mistake if the model makes the same mistake both ways, which is exactly what happened with the mint one.

**Problems with our own checker:** sometimes it says a number is wrong when it's actually fine, because the number got written as a word (Arabic الثانية for "2") or as "10 thousand" (10 آلاف). So the real number accuracy is probably a little higher than what we got.

## What we'd do next

1. **A "danger words" check:** a list of words that should never change in a school notice (allergy foods, "must", "not", days and times), checked both ways. This could catch stuff like nut → mint that back-translation misses.
2. **Try it with real parents:** use real notices from our school (with permission and no personal info), ask Urdu and Arabic speaking parents what they think, and add more languages people speak in the Gulf, like Bengali, Malayalam and Tagalog.

## How to run it yourself

Everything is in one notebook: `METranslate.ipynb`.

1. Make a free [Hugging Face](https://huggingface.co/join) account, then open the [FLORES+ page](https://huggingface.co/datasets/openlanguagedata/flores_plus) and accept its terms (the dataset is gated).
2. Create a **Read** token at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).
3. Open the notebook in [Google Colab](https://colab.research.google.com) (File → Upload notebook, or File → Open notebook → GitHub and paste this repo's link).
4. Go to **Runtime → Change runtime type → T4 GPU**.
5. Click the 🔑 key icon on the left, add a secret called `HF_TOKEN` with your token, and turn on **Notebook access**.
6. **Runtime → Run all.** It takes about 15–20 minutes. The demo app shows up near the end with a link you can open.

To test other languages, change `HIGH_LANG` / `LOW_LANG` in the settings cell (for example `pbt_Arab` for Pashto or `swh_Latn` for Swahili).

## Sources

- Dataset: [FLORES+](https://huggingface.co/datasets/openlanguagedata/flores_plus) (Open Language Data Initiative), devtest split
- Model: [NLLB-200 distilled 600M](https://huggingface.co/facebook/nllb-200-distilled-600M) (Meta)
- Metric: [sacrebleu](https://github.com/mjpost/sacrebleu) (chrF)
- Libraries: [Hugging Face Transformers](https://github.com/huggingface/transformers), [Datasets](https://github.com/huggingface/datasets), pandas, NumPy, SciPy, matplotlib, [Gradio](https://www.gradio.app)
- Tools: Google Colab (T4 GPU)
- AI help: we used Claude (made by Anthropic) to explain stuff we hadn't learned in school yet (python topics), help us code the statistical picture outputs and for debugging. We coded and ran all the experiments and did all the human checks ourselves, and we can explain every part of the code.

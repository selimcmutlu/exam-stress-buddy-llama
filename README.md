# Exam-Stress Buddy (Selim): Llama-3.2-3B fine-tuned for exam stress

My individual part of the group project *Exam-Stress Buddy* for the course Core Topics AI (assignment: LLM for human-artificial agent interaction).

Exam-Stress Buddy is a text chatbot for university students during exams. It notices how the student feels, helps with study planning, and refers the student to a counsellor or a crisis line when there is a risk.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/selimcmutlu/exam-stress-buddy-llama/blob/main/selim_exam_stress_buddy.ipynb)

Other group repositories: [k-noh/exam-stress-buddy](https://github.com/k-noh/exam-stress-buddy) (Qwen3-4B) and [DaniMihai95/CTAI_LLM](https://github.com/DaniMihai95/CTAI_LLM) (Qwen3.5-4B).

## What I did

- **Model:** [Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct). It is a different model family from my teammates' (Qwen), and small enough for a free Colab GPU.
- **Fine-tuning:** QLoRA (4-bit model, LoRA rank 16, 1 epoch) with [Unsloth](https://github.com/unslothai/unsloth), about 10 minutes per model.
- **Data:** 1,500 [EmpatheticDialogues](https://github.com/facebookresearch/EmpatheticDialogues) conversations about negative emotions, plus 24 exam-stress dialogues I wrote myself (`data/exam_examples.jsonl`).
- **Chain of thought:** before each reply, the model writes a hidden analysis of the student's emotion, need, risk and the strategy to use.
- **Safety:** a keyword check outside the model hands the student over to a counsellor or the 113 crisis line.

I compared five versions:

- `base`: the original model with a system prompt
- `base_cot`: `base` with chain of thought
- `ft_ed`: fine-tuned on EmpatheticDialogues
- `ft_mix`: fine-tuned on EmpatheticDialogues and my exam dialogues
- `ft_mix_cot`: `ft_mix` with chain of thought

## How I tested it

- Two six-turn scenarios shared with the group: S1 (night-before panic) and S2 (escalating distress).
- Four crash tests: risk without keywords, medication (Adderall), prompt injection, and "are you a real person?".
- Perplexity on 150 held-out EmpatheticDialogues conversations.
- I read every reply by hand. The results are in [`results/manual_review.md`](results/manual_review.md).

## Results

- Fine-tuning lowered perplexity from 65.8 to 12.4, but it made the bot worse: replies got short (83 words on average down to 12–24), it agreed to give exam answers, and it promised to keep self-harm a secret.
- My teammates saw the same with Qwen, so this is not about one model or one dataset.
- My 24 exam dialogues did not fix it.
- Chain of thought made the model's risk reading visible, but it missed indirect risk and made prompt injection easier.
- The base model with a good system prompt worked best. It noticed distress one turn before the keyword check did.
- Nothing caught risk when the student used no risk keywords.

**Conclusion:** a lower perplexity did not mean a better or safer helper, and safety checks should stay outside the model.

## How to run

1. Open the notebook in Google Colab and choose *Runtime → Change runtime type → T4 GPU*.
2. Choose *Runtime → Run all*. The full run takes about 60–80 minutes. For a 15-minute test, set `QUICK_MODE = True` in the *Configuration* cell first. The notebook here is set to `False` because it shows the outputs of the full run.
3. Part F downloads the results, and Part G starts a chat demo.

Tested with Colab (Tesla T4), Unsloth 2026.11.9, Transformers 5.17.0 and PyTorch 2.14.1. If a newer Unsloth breaks the notebook, run `%pip install unsloth==2026.11.9`.

The trained model (97 MB) is not in this repository. The notebook rebuilds it in about 10 minutes.

## Files

```
selim_exam_stress_buddy.ipynb   the whole pipeline, with the outputs of the full run
data/exam_examples.jsonl        my 24 exam-stress dialogues
scenarios/scenarios.json        the test scenarios and crash tests
results/                        transcripts, automatic checks and the manual review
```

## Limitations

- Few scenarios, one reply per turn, and no human ratings.
- EmpatheticDialogues was written by crowd workers, not counsellors.
- The keyword check misses indirect risk.
- This is a research prototype, not a mental-health tool.

## Use of AI tools

- Claude Code (Anthropic) helped me write, debug and document the code.
- Grammarly corrected spelling and grammar mistakes.
- Google Colab ran the fine-tuning and the evaluation.
- The research question, the design, the exam dialogues, running the experiments and the manual review are my own work.

## License

The code and the exam dialogues are under the [MIT License](LICENSE). EmpatheticDialogues (CC BY-NC 4.0) and Llama 3.2 (Llama 3.2 Community License) keep their own licenses and are not included here. Scenarios S1 and S2 come from the group repository [DaniMihai95/CTAI_LLM](https://github.com/DaniMihai95/CTAI_LLM).

## References

- Dettmers, T., Pagnoni, A., Holtzman, A., & Zettlemoyer, L. (2023). QLoRA: Efficient finetuning of quantized LLMs. *NeurIPS*.
- Grattafiori, A., et al. (2024). The Llama 3 herd of models. arXiv:2407.21783.
- Hu, E. J., et al. (2022). LoRA: Low-rank adaptation of large language models. *ICLR*.
- Qi, X., et al. (2024). Fine-tuning aligned language models compromises safety, even when users do not intend to! *ICLR*.
- Rashkin, H., Smith, E. M., Li, M., & Boureau, Y.-L. (2019). Towards empathetic open-domain conversation models: A new benchmark and dataset. *ACL*.
- Wei, J., et al. (2022). Chain-of-thought prompting elicits reasoning in large language models. *NeurIPS*.

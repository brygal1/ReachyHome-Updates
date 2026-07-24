# Reachy Home 1.0.19

Fix conversation quality: narration, Qwen first-synth, motor-noise barge-in (#139)

Stop spoken stage directions, repair the Qwen first-synth/STT regression, cut first-token latency (drop low-effort thinking), and stop motor noise self-interrupting the robot. Hardened over two adversarial review passes (classify-only narration filter that never mutates spoken content; barge-in teardown/restart safety). ruff + mypy clean; 3028 unit+contract tests pass.

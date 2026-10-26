# Reachy Home 1.0.135

Recognize speakers within a sentence on live robot-mic turns (#311)

Live turns score well below the enrollment clips the speaker policy was tuned
on (real speech 0.34-0.50), so the short-turn penalty, the extra prefix
headroom and the two-turn confirmation held identification back until a long
sentence ended. In 60 sampled live checks only 4 were accepted, and the
retained speaker stayed "Bart" while Holly spoke.

The policy now uses one flat cosine floor (0.38) at every duration and a
runner-up margin of 0.10. A lead of 0.15 or more is decisive: only a decisive
lead accepts a prefix, and a decisive accepted match commits in one turn. A
narrower final match still needs the configured confirmations, and a
rejection never decides alone. The first prefix check moves to 1.5 s.

The proposed 0.33 floor failed ADR-0062's gate: leave-one-out accepted one
consented out-of-household voice at 0.367. 0.38 passes both private passes
with zero cross-person and zero unknown accepts. Recognition at 2 s improves
from 4/23 to 15/23 (leave-one-out) and from 3/14 to 7/14 (session split).
Reachy Home's training guidance follows the same floor and margin, and a
contract test now locks its fixture to the configured policy.

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>

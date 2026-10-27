# Reachy Home 1.0.139

Let the Memory Brain wait for Reachy Home's Body grant instead of failing (#315)

After a restart, or while the Mac sleeps, there is no Body grant. A
session dream now releases its run and retries like a preemption, and
tonight's reflection keeps retrying instead of being lost until the next
night. Reflect Now reports the missing grant plainly.

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>

# Reachy Home 1.0.52

Parallelize general CI across three runners (#206)

Add two local self-hosted runner services and route general CI through a shared reachy-general pool while keeping Apple/Xcode jobs serialized on the primary runner.

Update workflow contracts, documentation, and coverage worker configuration for the three-runner topology.

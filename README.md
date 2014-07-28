Summary of Diego AZ Balancing Spike
===================================

I spiked on implementing [the AZ balancing feature](https://www.pivotaltracker.com/story/show/70403224).  Herein lies the summary of the spike.  I'm putting this in a repo instead of a gist so it can have collaborators.

1. <a href="#main-goal">Main Goal</a>
2. <a href="#other-wins">Other Wins</a>
3. <a href="#completed-tasks">Completed Tasks</a>
4. <a href="#non-goals">Non-Goals</a>
5. <a href="#remaining-work">Remaining Work</a>

## <a id="main-goal"/>Main Goal

> Start and Stop Auctions should take AZ (or Cluster) into account with multi-instance apps.  Instances should be well-balanced across AZs so that the High Availability/Fault Tolerance a user gets by having multiple instances is less brittle in the face of an AZ going down.  In that regard, this is to improve our SLA.

## <a id="other-wins"/>Other Wins

* Add useful auction simulations (including some simulations are not AZ-related)
* Add some sorely-needed variance statistics for existing variables in the simulation report
* Add statistics for some important variables not previously covered in the simulation report (including many variables that are not AZ-related)
* Make simulation report generation code more sane
* Make simulation setup code more sane
* Bring a little more consistency between treatment of StopAuctions vis-à-vis StartAuctions where it improves sanity

## <a id="completed-tasks"/>Completed Tasks

* Refactor simulation so its easy to add more tests. [Auction](https://github.com/amitkgupta/auction/commit/385fa9c5c079e3e3675d62fb69d21d6af44c1350)
* Give Reps knowledge of what "AZ" they're on, and add statistics about AZ balancing *before* implementing AZ balancing, so we can see the before/after improvement. [Rep](https://github.com/amitkgupta/rep/commit/07d0d79ec73a955b5f2703381dbb31971a213929) [Auction](https://github.com/amitkgupta/auction/commit/1fcc123864903bc61aa2b2996d87df55f63a85f8)
* App-Manager knows how many AZs Reps/Executors are distributed across, and communicates this number to the Auctioneer via LRPStartAuction and LRPStopAuction. [App-Manager](https://github.com/amitkgupta/app-manager/commit/4ff9629818699c04cb2095ebe0558be9eb390dec) [Auctioneer](https://github.com/amitkgupta/auctioneer/commit/e312a4c7cad6d2a29b5b88a9b582c7362b7e1f9e) [Runtime-Schema](https://github.com/amitkgupta/runtime-schema/commits/az_balance)
* Auction combines NumberOfAZs from App-Manager and AZNumber from Rep to take AZ balancing into account when computing score/bid. [Auction](https://github.com/amitkgupta/auction/commit/3e946ca0887934fb5da3efdf63e79a810b696468)
* Update Inigo, see it pass. [Inigo](https://github.com/amitkgupta/inigo/commit/24a8e06f13ad0110dd1d45bf3036aadaf38e5c9c)
* Bump Deps. [App-Manager](https://github.com/amitkgupta/app-manager/commit/2caa83c41ec3db33b1faf6520e289140c09c4437) [Auctioneer](https://github.com/amitkgupta/auctioneer/commit/f54fbb54a8851375d55215bcb450c79160324b50) [Inigo](https://github.com/amitkgupta/inigo/commit/a49fd2686f1a7c5235311072264b6760fbc1320d) [Rep](https://github.com/amitkgupta/rep/commit/d8c889466c90df6f8d32b9062aa4b5cdafacade1)
* Update Diego-Release to pass `numAZs` to App-Manager and `AZNumber` to each rep. [Diego-Release](https://github.com/amitkgupta/diego-release/commits/az_balance)
* BOSH-Lite-AWS deploy `cf-release/acceptance-deployed` and `diego-release/az_balance` and see CATS pass. I've done this, and have an EC2 jumpbox you can go on and run CATS (and Inigo).

## <a id="non-goals"/>Non-Goals

* Pre-filter Reps out of the auction according to whether they belong to / don't belong to a given Placement Pool / Tag
  **BUT**: this will be easy to add on, and it's entirely separate since it's pre-filtering, not part of the main run of the auction
* Ensure that we can always handle apps with large memory or disk requirements (i.e. ensure that we don't distribute the small granular "sand" apps so well that we have no room for the "boulders")
  **BUT**: a simulation has been added to try and capture this
* Fine-tune auction parameters to fully optimize app placement behaviour
  **BUT**: it's totally straightforward to tweak the coefficients for [Start](https://github.com/amitkgupta/auction/blob/az_balance/auctionrep/auction_rep.go#L266-L270) and [Stop](https://github.com/amitkgupta/auction/blob/az_balance/auctionrep/auction_rep.go#L290-L293) Auctions
* Make auction parameters configurable via the BOSH deployment manifest
  **BUT**: this would be very easy to do if desired
* Speed up time between a user pushing an app and it being started, e.g. by taking into account whether a Rep already has the droplet for the app cached locally
  **BUT**: performance is just as good as before after this spike, and these performance enhancements can be added totally separately later

## <a id="remaining-work"/>Remaining Work

* Right now only in-process simulation reps are balanced across multiple "AZs", the corresponding work needs to be done when `communicationMode` is something other than "in-process", e.g. NATS.
* SVG (browser) reports for simulation have not been touched.  All the improved statistics are in the CLI report. SVG reports need some love.
* Merge work onto `master` (or `develop` in the case of `diego-release`).


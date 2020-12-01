# Properties of Lucky 7


### Lucky 7 Property: Operation Safe, Optimizable

Uses `output_range` for its write operations:

- safe by default (three-legged algorithms + range-v3 are not!)
	- `output_range` stops the user if they reach end-of-range
	- Requires explicit "infinity range" opt-in (`unbounded_view`)
- optimized `std` use iterators/ranges (not output sinks)


### Lucky 7 Property: Low, Bounded Memory Usage

Algorithms will not consume more than:

- iterator/view temporaries (`span`, etc. copies)
- basic temporaries (booleans, size book keeping, etc.)
- `2 ✖ max_code_points` + `2 ✖ max_code_units`

Basically: all stack-based memory (no allocations!)


### Lucky 7 Property: Arbitrary Encodings

Can connect (`transcode`) arbitrary encodings:

- as long as they share a common/convertible `code_point` type
- as long as they have the 7 basis operations


### Lucky 7 Property: Scalable

7 basis operations unlock the entire set of functionality!

- Design does not degrade with time (important for `std::`)
- Committee does not control what encodings are allowed
- Companies do not control what encodings are worth it


### Lucky 7 Property: Speed

...

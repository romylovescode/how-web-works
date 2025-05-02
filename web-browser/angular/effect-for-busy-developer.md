### LInks
- https://angular.love/signals-in-angular-deep-dive-for-busy-developers

Signals as primitives
A signal represents a cell of data which may change over time. Signals may be either “state” (just a value which is set manually) or “computed” (think of a formula based on other signals).

Computed signals function by automatically tracking which other signals are read during their evaluation. When a computed signal is read, it checks whether any of its previously recorded dependencies have changed, and re-evaluates itself if so.

For example, here we have a state signal counter and a computed signalisEven. We set the initial value for counter to 0 and later change it to 1. You can see that the computed signal isEven reacts to the changes by producing two different values before and after the update of the counter signal:

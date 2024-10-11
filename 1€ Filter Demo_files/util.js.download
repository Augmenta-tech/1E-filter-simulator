// ===================
// = Throttled calls =
// ===================
// An object that will call a function F exactly X seconds after the first use of .call(), no matter how many subsequent uses of .call() before the actual execution of F.

function ThrottledCall(fn, time) {
	this.fn = fn
	this.time = time
	this.timer = null
}
ThrottledCall.prototype.callFn = function() {
	this.fn()
	this.timer = null
}
ThrottledCall.prototype.cancel = function() {
	clearTimeout(this.timer)
	this.timer = null
}
ThrottledCall.prototype.call = function() {
	if(this.timer === null) {
        // this.fn()
        this.timer = setTimeout(this.callFn.bind(this), this.time)
	} else {}
}
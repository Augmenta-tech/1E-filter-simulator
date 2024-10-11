(function() {
	function mean(x) {
		var sum = 0;
		for(var i=0; i<x.length; ++i) {
			sum += x[i];
		}
		return sum / x.length;
	}
	
	function median(x) {
		x.sort(function(a,b) { return a-b; });
		var half = Math.floor(x.length / 2);
		if(x.length % 2) {
			return x[half];
		}
		return (x[half - 1] + x[half]) / 2;
	}

	function MovingWindowFilter(windowSize) {
		this.windowSize = windowSize;
		this.filteringFunction = mean;
		this.window = [];
		
		return this;
	}
	MovingWindowFilter.description = "Moving average";
	MovingWindowFilter.parameters = [
		{ name: 'windowSize', description: 'Window size', min: 1, max: Infinity, def: 14 }
	];
	MovingWindowFilter.prototype.filter = function(x) {
		if(this.window.length < this.windowSize) {
			this.window.push(x);
		} else {
			this.window.shift();
			this.window.push(x);
		}
		return this.filteringFunction(this.window);
	};
	
	function SingleExponentialFilter(alpha, variant) {
		this.alpha = alpha;
		this.variant = variant || 'predict';
		this.lastEstimate = undefined;
		this.lastX = undefined;
		return this;
	}
	SingleExponentialFilter.description = "Single exponential";
	SingleExponentialFilter.parameters = [
		{ name: 'alpha', description: 'Alpha', min: 0.0, max: 1.0, def: 0.11, step: 0.01 },
		// { name: 'variant', description: 'Variant', values: ['predict', 'lowpass'], def: 'predict', optional: true }
	];
	SingleExponentialFilter.prototype.filter = function(x) {
		if(this.lastX === undefined) {
			this.lastEstimate = x;
		} else {
			this.lastEstimate = this[this.variant](x);
		}
		this.lastX = x;
		
		return this.lastEstimate;
	};
	SingleExponentialFilter.prototype.predict = function(x) {
		return this.alpha * this.lastX + (1.0 - this.alpha) * this.lastEstimate;
	};
	SingleExponentialFilter.prototype.lowpass = function(x) {
		return this.alpha * x + (1.0 - this.alpha) * this.lastEstimate;
	};
	
	function DoubleExponentialFilter(alpha, gamma) {
		this.alpha = alpha;
		this.gamma = gamma;
		this.lastX = undefined;
		this.lastEstimate = undefined;
		this.b = 0;
		return this;
	}
	DoubleExponentialFilter.description = "Double exponential";
	DoubleExponentialFilter.parameters = [
		{ name: 'alpha', description: 'Alpha', min: 0.0, max: 1.0, def: 0.11, step: 0.01 },
		{ name: 'gamma', description: 'Gamma', min: 0.0, max: 1.0, def: 0.11, step: 0.01 },
	];
	DoubleExponentialFilter.prototype.filter = function(x) {
		if(this.lastX === undefined) {
			this.lastEstimate = x;
			this.b = 0;
		} else {
			var s = this.alpha * x + (1.0 - this.alpha) * (this.lastEstimate + this.b);
			this.b = this.gamma * (s - this.lastEstimate) + (1.0 - this.gamma) * this.b;
			this.lastEstimate = s;
		}
		this.lastX = x;
		
		return this.lastEstimate;
	};
	
	function DESPFilter(alpha, tau) {
		this.alpha = alpha;
		this.tau = tau;
		this.hatxiprev = undefined;
		this.hatxi2prev = undefined;
		return this;
	}
	DESPFilter.description = "Double exponential";
	DESPFilter.parameters = [
		{ name: 'alpha', description: 'Alpha', min: 0.0, max: 1.0, def: 0.06, step: 0.01 },
		{ name: 'tau', description: 'Tau', min: 1, max: Infinity, def: 1, optional: true },
	];
	DESPFilter.prototype.filter = function(x) {
		if(this.hatxiprev === undefined) {
			this.hatxiprev = x;
		} else {
			this.hatxiprev = this.alpha * x + (1.0 - this.alpha) * this.hatxiprev;
		}
		if(this.hatxi2prev === undefined)  {
			this.hatxi2prev = this.hatxiprev;
		} else {
			this.hatxi2prev = this.alpha * this.hatxiprev + (1.0 - this.alpha) * this.hatxi2prev;
		}
		
		return (2.0 + (this.alpha * this.tau) / (1.0 - this.alpha)) * this.hatxiprev - (1.0 + (this.alpha * this.tau) / (1.0 - this.alpha)) * this.hatxi2prev;
	};
	
	function LinearKalmanFilter(A, B, H, x, P, Q, R) {
		this.A = A;
		this.B = B;
		this.H = H;
		this.currentStateEstimate = x;
		this.currentProbEstimate = P;
		this.Q = Q;
		this.R = R;
		return this;
	}
	LinearKalmanFilter.prototype.step = function(controlVector, measurementVector) {
		var tmp = this.A.x(this.currentStateEstimate);
		var predictedStateEstimate = tmp.add(this.B.x(controlVector));
		var predictedProbEstimate = this.A.x(this.currentProbEstimate).x(this.A.transpose()).add(this.Q);
		
		var innovation = this.H.x(measurementVector).subtract(predictedStateEstimate);
		var innovationCovariance = this.H.x(predictedProbEstimate).x(this.H.transpose()).add(this.R);
		
		var kalmanGain = undefined;
		if(innovationCovariance.rows() == innovationCovariance.cols() && innovationCovariance.rows() == 1) {
			// bugfix for sylvester.js
			var invInnovCov = 1.0 / innovationCovariance.e(1, 1);
			kalmanGain = predictedProbEstimate.x(this.H.transpose()).x($M([[invInnovCov]]));
		} else {
			kalmanGain = predictedProbEstimate.x(this.H.transpose()).x(innovationCovariance.inverse());
		}
		this.currentStateEstimate = predictedStateEstimate.add(kalmanGain.x(innovation));
		this.currentProbEstimate = Matrix.I(this.currentProbEstimate.rows()).subtract(kalmanGain.x(this.H)).x(predictedProbEstimate);
	}
	
	function ConstantValueKalmanFilter(p, q, r) {
		this.p = p;
		this.q = q;
		this.r = r;
		
		this.kalmanFilter = undefined;
		this.controlVector = $M([[0]]);
		
		return this;
	}
	ConstantValueKalmanFilter.description = "Kalman filter";
	ConstantValueKalmanFilter.parameters = [
		{ name: 'p', description: 'Initial covariance estimate', def: 1.0, optional: true },
		{ name: 'q', description: 'Process error covariance', def: 0.3, step: 0.01 },
		{ name: 'r', description: 'Measurement error covariance', def: 18.06, step: 0.1 }
	];
	ConstantValueKalmanFilter.prototype.filter = function(x) {
		if(this.kalmanFilter === undefined) {
			this.kalmanFilter = new LinearKalmanFilter($M([[1]]), $M([[0]]), Matrix.I(1), $M([[x]]), $M([[this.p]]), $M([[this.q]]), $M([[this.r]]));
			return x;
		}
		this.kalmanFilter.step(this.controlVector, $M([[x]]));
		return this.kalmanFilter.currentStateEstimate.e(1, 1);
	}
	
	function OneEuroFilter(freq, mincutoff, beta, dcutoff) {
		this.freq = freq;
		this.mincutoff = mincutoff;
		this.beta = beta;
		this.dcutoff = dcutoff;
		
		this.x = new SingleExponentialFilter(this.alpha(this.mincutoff), 'lowpass');
		this.dx = new SingleExponentialFilter(this.alpha(this.dcutoff), 'lowpass');
		this.lastTime = undefined;
		
		return this;
	}
	OneEuroFilter.description = "1€ Filter";
	OneEuroFilter.parameters = [
		{ name: 'freq', description: 'Frequency', min: 1.0, max: Infinity, def: 25, optional: true },
		{ name: 'mincutoff', description: 'fcmin', min: 0.0, max: 10, def: 1.0, step: 0.01, logstep: true },
		{ name: 'beta', description: 'beta   ', def: 0.007, min: 0, max: 1.0, step: 0.001 },
		{ name: 'dcutoff', description: 'Cutoff for derivative', min: 0, max: 10.0, def: 1.0, optional: false, step: 0.01 },
	];
	OneEuroFilter.prototype.alpha = function(cutoff) {
		var te = 1.0 / this.freq;
		var tau = 1.0 / (2 * Math.PI * cutoff);
		return 1.0 / (1.0 + tau/te);
	};
	OneEuroFilter.prototype.filter = function(x, timestamp) {
		if(this.lastTime !== undefined && timestamp !== undefined) {
			this.freq = 1.0 / (timestamp - this.lastTime);
		}
		this.lastTime = timestamp;
		// Fixed in August 2023
		var previousX = this.x.lastEstimate;
		var dx = (previousX === undefined)? 0 : (x - previousX) * this.freq;
		this.dx.alpha = this.alpha(this.dcutoff);
		var edx = this.dx.filter(dx);
		var cutoff = this.mincutoff + this.beta * Math.abs(edx);
		this.x.alpha = this.alpha(cutoff);
		return this.x.filter(x);
	}
	
	window.Filters = {
		MovingWindowFilter: MovingWindowFilter,
		SingleExponentialFilter: SingleExponentialFilter,
		// DoubleExponentialFilter: DoubleExponentialFilter,
		DESPFilter: DESPFilter,
		ConstantValueKalmanFilter: ConstantValueKalmanFilter,
		OneEuroFilter: OneEuroFilter
	};
})();
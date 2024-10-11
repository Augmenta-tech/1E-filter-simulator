(function() {
	/* ECMA-262 5TH EDITION COMPATIBILITY FIX */
	if (!Function.prototype.bind) {
		Function.prototype.bind = function (oThis) {
			if (typeof this !== "function") {
				// closest thing possible to the ECMAScript 5 internal IsCallable function
				throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
			}

			var aArgs = Array.prototype.slice.call(arguments, 1), 
			fToBind = this, 
			fNOP = function () {},
			fBound = function () {
				return fToBind.apply(this instanceof fNOP
					? this
					: oThis || window,
					aArgs.concat(Array.prototype.slice.call(arguments)));
			};

			fNOP.prototype = this.prototype;
			fBound.prototype = new fNOP();

			return fBound;
		};
	}
	/* */
	/////////////////////////////////////////////////////////////////////

	
	var $$$$ = window.$$$$ = function(value, validationFunction, options) {
		if(typeof(value) !== typeof(undefined)) {
			if(value instanceof $$$$.Ref) {
				if(validationFunction instanceof $$$$.Ref) {
					// Create a binding
					return new $$$$.Binding(value, validationFunction, options);
				} else {
					return value;
				}
			} else {
				return new $$$$.Ref(value, validationFunction);
			}
		}
		
		return this;
	};
	
	// ==============
	// = Observable =
	// ==============
	// Something that can be subscribed to for something.
	// Use subscribe() or unsusbcribe() to manage subscriptions.
	// Use notify() to notify all subscribers.
	
	$$$$.Observable = function() {
		this.observers = [];
	};
	
	$$$$.Observable.prototype.notify = function() {
		for(var i in this.observers) {
			this.observers[i].apply(this, arguments);
		}
	};
	
	$$$$.Observable.prototype.subscribe = function(fn) {
		this.observers.push(fn);
	};
	
	$$$$.Observable.prototype.unsubscribe = function(fn) {
		var i = this.observers.indexOf(fn);
		if(i >= 0)
			this.observers.splice(i, 1);
	};
	
	$$$$.O = function() {
		return new $$$$.Observable();
	}
	
	// ========================
	// = Validation functions =
	// ========================
	// Some commonly used validation functions
	
	$$$$.ValidationFunctions = {
		identity: function(x) { return x; }
	};
	
	// =======
	// = Ref =
	// =======
	// A Ref holds a value. Subscribers can be notified when the value changes.
	// The value can be accessed with value.
	
	$$$$.Ref = function(value, validationFunction) {
		$$$$.Observable.call(this);
		this._value = value;
		this.validate = validationFunction || $$$$.ValidationFunctions.identity;
	};
	
	$$$$.Ref.prototype = new $$$$.Observable();
	$$$$.Ref.prototype.constructor = $$$$.Ref;
	
	$$$$.Ref.prototype.update = function() {
		this.value = this.value;
	};
	
	$$$$.Ref.prototype.get = function() {
		return this._value;
	};
	
	$$$$.Ref.prototype.set = function(x, info) {
		var formerValue = this.value;
		this._value = this.validate(x);
		this.notify(this._value, formerValue, info);
	};
	$$$$.Ref.prototype.propagate = function() {
		this.notify(this._value, this._value, undefined);
	}
	
	Object.defineProperty($$$$.Ref.prototype, 'value', {
		enumerable: true,
		get: function() {
			return this.get();
		},
		set: function(x, info) {
			this.set(x, info);
		}
	});
	
	$$$$.R = function(v, vF) {
		return new $$$$.Ref(v, vF);
	}
	
	// $$$$ TODO: here
	
	// ===========
	// = Binding =
	// ===========
	// A constraint that binds two Refs, a and b.
	// Use options aToB and bToA to define conversion functions.
	// Use options a$silent and b$slient to define the direction of the binding.
	
	$$$$.Binding = function(a, b, options) {
		var _a = undefined;
		var _b = undefined;
		
		if(typeof(options) === typeof(undefined))
			options = {};
		
		this.a$silent = options.a$silent;
		this.b$silent = options.b$silent;
		this.aToB = options.aToB || $$$$.ValidationFunctions.identity;
		this.bToA = options.bToA || $$$$.ValidationFunctions.identity;
		
		Object.defineProperty(this, 'a', {
			get: function() {
				return _a;
			},
			set: function(x) {
				if(this.a)
					this.a.unsubscribe(this.aDidChange.bind(this));
				_a = x;
				if(this.a)
					this.a.subscribe(this.aDidChange.bind(this));
			}
		});
		
		Object.defineProperty(this, 'b', {
			get: function() {
				return _b;
			},
			set: function(x) {
				if(this.b)
					this.b.unsubscribe(this.bDidChange.bind(this));
				_b = x;
				if(this.b)
					this.b.subscribe(this.bDidChange.bind(this));
			}
		});
		
		this.a = a;
		this.b = b;
	};
	
	$$$$.Binding.prototype.aDidChange = function(x, oldX, info) {
		if(!this.a$silent && this.b && info != this) {
			this.b.set(this.aToB(x), this);
		}
	};
	
	$$$$.Binding.prototype.bDidChange = function(x, oldX, info) {
		if(!this.b$silent && this.a && info != this) {
			this.a.set(this.bToA(x), this);
		}
	};
	
	$$$$.B = function(a, b, o) {
		return new $$$$.Binding(a, b, o);
	}
	
	// ===================================
	// = Object.ref =
	// ===================================
	// Adds a method to all objects that creates a ref.
	// Facilitates manipulation of refs and bindings.
	
	Object.defineProperty(Object.prototype, 'ref', {
		value: function(name, defaultValue, validationFunction) {
			if(typeof(this[name]) !== typeof(undefined))
				return this[name];
			
			var _value = undefined;
			
			if(defaultValue instanceof $$$$.Ref) {
				_value = defaultValue;
			} else {
				_value = new $$$$.Ref(defaultValue, validationFunction);
			}
			
			var _bindings = [];
			
			Object.defineProperty(this, name, {
				enumerable: true,
				configurable: true,
				get: function() {
					return _value.value;
				},
				set: function(x) {
					_value.value = x;
				}
			});
			
			Object.defineProperty(this, name + "$subscribe", {
				enumerable: false,
				configurable: true,
				value: function(fn) {
					_value.subscribe(fn);
				}
			});
			
			Object.defineProperty(this, name + "$unsubscribe", {
				enumerable: false,
				configurable: true,
				value: function(fn) {
					_value.unsubscribe(fn);
				}
			});
			
			Object.defineProperty(this, name + "$ref", {
				enumerable: false,
				configurable: true,
				get: function() {
					return _value;
				}
			});
			
			Object.defineProperty(this, name + "$bindings", {
				enumerable: false,
				configurable: true,
				get: function() {
					return _bindings;
				}
			});
			
			Object.defineProperty(this, name + "$bind", {
				enumerable: false,
				configurable: true,
				value: function(b, options) {
					var binding = new $$$$.Binding(_value, b, options);
					this[name + '$bindings'].push(binding);
					return binding;
				}
			});
			
			return _value;
		}
	});
	
	// ===========================
	// = ExternalRef =
	// ===========================
	// Allows to treat diverse objects like Refs.
	/*
		Example on an <input> field with jQuery:
		
			var foo = $('input#foo');
			var fooValue = new $$$$.ExternalRef({
				get: function() { return foo.val(); },
				set: function() { return foo.val(x); },
				whenReady: function(that) {
					foo.change(function() { that.externalChanged(); });
				},
				validate: function(x) { return parseFloat(x); }
			});
	*/
	
	$$$$.ExternalRef = function(options) {
		if(!options)
			return;
		
		var get = options.get;
		var set = options.set;
		var whenReady = options.whenReady;
		var validate = options.validate;
		
		$$$$.Ref.call(this, get(), validate);
		this.externalGet = get;
		this.externalSet = set;
		
		if(whenReady)
			whenReady(this);
	};
	$$$$.ExternalRef.prototype = new $$$$.Ref();
	$$$$.ExternalRef.prototype.constructor = $$$$.ExternalRef;
	$$$$.ExternalRef.prototype.updateWith = function(ref) {
		var that = this;
		ref.subscribe(function() {
			that.externalChanged();
		});
	}
	$$$$.ExternalRef.prototype.externalChanged = function() {
		var v = this.validate(this.externalGet());
		
		if(this.get() != v) {
			this.set(v, this);
		}
	};
	$$$$.ExternalRef.prototype.get = function() {
		if(this.externalGet && !this.externalSet) {
			return this.externalGet();
		}
		return $$$$.Ref.prototype.get.call(this);
	}
	$$$$.ExternalRef.prototype.set = function(x, info) {
		$$$$.Ref.prototype.set.call(this, x, info);
		if(info != this && this.externalSet)
			this.externalSet(this.get());
	};
	
	$$$$.E = function(o) {
		return new $$$$.ExternalRef(o);
	}
	
	$$$$.ArrayElementRef = function(initialArray, initialIndex) {
		this.array = initialArray;
		this.ref('index', 0);
		if(typeof(initialIndex) === 'number')
			this.index = initialIndex;
		this.index$subscribe(this.indexChanged.bind(this));
		$$$$.Ref.call(this, this.array[this.index]);
		// TODO: 
	}
	$$$$.ArrayElementRef.prototype = new $$$$.Ref();
	$$$$.ArrayElementRef.prototype.constructor = $$$$.Ref;
	$$$$.ArrayElementRef.prototype.set = function(x, info) {
		$$$$.Ref.prototype.set.call(this, x, info);
		this.array[this.index] = this.value;
	};
	$$$$.ArrayElementRef.prototype.indexChanged = function(i) {
		$$$$.Ref.prototype.set.call(this, this.array[i]);
	}
	$$$$.ArrayElementRef.prototype.push = function(x) {
		this.array.push(x);
	}
	$$$$.ArrayElementRef.prototype.hasNext = function() {
		return this.index < this.length - 1;
	}
	Object.defineProperty($$$$.ArrayElementRef.prototype, 'length', {
		get: function() { return this.array.length; }
	});
	
	$$$$.AE = function() {
		return new $$$$.ArrayElementRef(arguments, 0);
	}
	
	// ==========================================
	// = jQuery ExternalRef helpers =
	// ==========================================
	// Allows to make Refs from DOM attributes quite easily.
	
	if(typeof(jQuery) !== typeof(undefined)) {
		if(jQuery.fn.watch) {
			$$$$.ObservableDOMAttribute = function(element, attribute, validationFunction) {
				var $element = this.$element = jQuery(element);
				
				$$$$.ExternalRef.call(this, {
					get: function() {
						return $element.attr(attribute);
					},
					set: function(x) {
						$element.attr(attribute, x);
					},
					whenReady: function(that) {
						$element.watch(attribute, function() {
							that.externalChanged();
						});
					},
					validate: validationFunction
				});
			};
			$$$$.ObservableDOMAttribute.prototype = new $$$$.ExternalRef();
			$$$$.ObservableDOMAttribute.prototype.constructor = $$$$.ObservableDOMAttribute;
	
			$$$$.DOM = function(e, a, vF) {
				return new $$$$.ObservableDOMAttribute(e, a, vF);
			}
			
			$$$$.JQueryFunctionFromDOMAttribute = function(element, fnName, attributeToWatch, validationFunction) {
				var $element = this.$element = jQuery(element);
				
				var initFn = function() {};
				if(attributeToWatch) {
					initFn = function(that) {
						$element.watch(attributeToWatch, function() {
							that.externalChanged();
						});
					}
				}
				
				$$$$.ExternalRef.call(this, {
					get: function() {
						return $element[fnName]();
					},
					set: function(x) {
						$element[fnName](x);
					},
					whenReady: initFn,
					validate: validationFunction
				});
			};
			$$$$.JQueryFunctionFromDOMAttribute.prototype = new $$$$.ExternalRef();
			$$$$.JQueryFunctionFromDOMAttribute.prototype.constructor = $$$$.JQueryFunctionFromDOMAttribute;			
	
			$$$$.JQDOM = function(e, fN, aTW, vF) {
				return new $$$$.JQueryFunctionFromDOMAttribute(e, fN, aTW, vF);
			}
		}
	}
})();
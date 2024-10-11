(function($) {
	$.fn.disableSelection = function() {
        return this.attr('unselectable', 'on').css('user-select', 'none').css('-moz-user-select', 'none').css('-khtml-user-select', 'none').css('-webkit-user-select', 'none').on('selectstart', false);
    };
	
	$.fn.tanglify = function(options) {
		var $that = this;
		
		var settings = $.extend({
			valueInsideSelector: '.tanglify_value',
			updateFn: function() {},
			min: -Infinity,
			max: Infinity,
			step: 1,
			defaultValue: 50
		}, options);
		
		var hovered = false;
		var down = false;
		var editing = false;
		var value = settings.defaultValue;
		$(settings.valueInsideSelector, $that).attr({
			max: settings.max
			,min: settings.min
			,step: settings.step
		})
		//nice
		function updateValue() {
			if(!editing) {
				//$(settings.valueInsideSelector, $that).text(value.toPrecision(3))
				$(settings.valueInsideSelector, $that).val(value.toPrecision(3))
			}
		}
		
		var lastPageX = undefined;

		this.find(settings.valueInsideSelector).change(function (e) {
			value = +$(this).val();
			updateValue();
			settings.updateFn(value);
		})
		this.find('.tanglify_reset').on('click', function (e) {
			value = settings.defaultValue;
			updateValue();
			settings.updateFn(value);
		})

		updateValue();
		
		return this;
	};
})(jQuery);
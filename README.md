# mobileNativeJS
# 学习妙味课堂的《移动端原生技法精粹揭秘课程》时，记录的一些笔记

----------

### requestAnimationFrame和cancelAnimationFrame的兼容处理

	(function(){
		window.requestAnimationFrame = window.requestAnimationFrame||window.webkitRequestAnimationFrame||window.mozRequestAnimationFrame;
		window.cancelAnimationFrame = window.cancelAnimationFrame|| window.webkitCancelAnimationFrame || window.mozCancelAnimationFrame||window.cancelRequestAnimationFrame||window.webkitCancelRequestAnimationFrame||window.mozCancelRequestAnimationFrame; 
		if(!requestAnimationFrame){
			var lastTime = Date.now();
			window.requestAnimationFrame = function(callback){
				var id;
				var nowTime = Date.now();
				var delay = Math.max(16.7-(nowTime-lastTime),0);
				id = setTimeout(callback,delay);
				lastTime = nowTime + delay;
				return id;
			};
		}
		if(!cancelAnimationFrame){
			window.cancelAnimationFrame = function(index){
				clearTimeout(index);
			};
		}
	})();

### Tween.js
	var Tween = {
		linear: function (t, b, c, d){
			return c*t/d + b;
		},
		easeIn: function(t, b, c, d){
			return c*(t/=d)*t + b;
		},
		easeOut: function(t, b, c, d){
			return -c *(t/=d)*(t-2) + b;
		},
		easeBoth: function(t, b, c, d){
			if ((t/=d/2) < 1) {
				return c/2*t*t + b;
			}
			return -c/2 * ((--t)*(t-2) - 1) + b;
		},
		easeInStrong: function(t, b, c, d){
			return c*(t/=d)*t*t*t + b;
		},
		easeOutStrong: function(t, b, c, d){
			return -c * ((t=t/d-1)*t*t*t - 1) + b;
		},
		easeBothStrong: function(t, b, c, d){
			if ((t/=d/2) < 1) {
				return c/2*t*t*t*t + b;
			}
			return -c/2 * ((t-=2)*t*t*t - 2) + b;
		},
		elasticIn: function(t, b, c, d, a, p){
			if (t === 0) { 
				return b; 
			}
			if ( (t /= d) == 1 ) {
				return b+c; 
			}
			if (!p) {
				p=d*0.3; 
			}
			if (!a || a < Math.abs(c)) {
				a = c; 
				var s = p/4;
			} else {
				var s = p/(2*Math.PI) * Math.asin (c/a);
			}
			return -(a*Math.pow(2,10*(t-=1)) * Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
		},
		elasticOut: function(t, b, c, d, a, p){
			if (t === 0) {
				return b;
			}
			if ( (t /= d) == 1 ) {
				return b+c;
			}
			if (!p) {
				p=d*0.3;
			}
			if (!a || a < Math.abs(c)) {
				a = c;
				var s = p / 4;
			} else {
				var s = p/(2*Math.PI) * Math.asin (c/a);
			}
			return a*Math.pow(2,-10*t) * Math.sin( (t*d-s)*(2*Math.PI)/p ) + c + b;
		},    
		elasticBoth: function(t, b, c, d, a, p){
			if (t === 0) {
				return b;
			}
			if ( (t /= d/2) == 2 ) {
				return b+c;
			}
			if (!p) {
				p = d*(0.3*1.5);
			}
			if ( !a || a < Math.abs(c) ) {
				a = c; 
				var s = p/4;
			}
			else {
				var s = p/(2*Math.PI) * Math.asin (c/a);
			}
			if (t < 1) {
				return - 0.5*(a*Math.pow(2,10*(t-=1)) * 
						Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
			}
			return a*Math.pow(2,-10*(t-=1)) * 
					Math.sin( (t*d-s)*(2*Math.PI)/p )*0.5 + c + b;
		},
		backIn: function(t, b, c, d, s){
			if (typeof s == 'undefined') {
			   s = 1.70158;
			}
			return c*(t/=d)*t*((s+1)*t - s) + b;
		},
		backOut: function(t, b, c, d, s){
			if (typeof s == 'undefined') {
				s = 2.70158;  //回缩的距离
			}
			return c*((t=t/d-1)*t*((s+1)*t + s) + 1) + b;
		}, 
		backBoth: function(t, b, c, d, s){
			if (typeof s == 'undefined') {
				s = 1.70158; 
			}
			if ((t /= d/2 ) < 1) {
				return c/2*(t*t*(((s*=(1.525))+1)*t - s)) + b;
			}
			return c/2*((t-=2)*t*(((s*=(1.525))+1)*t + s) + 2) + b;
		},
		bounceIn: function(t, b, c, d){
			return c - Tween['bounceOut'](d-t, 0, c, d) + b;
		},       
		bounceOut: function(t, b, c, d){
			if ((t/=d) < (1/2.75)) {
				return c*(7.5625*t*t) + b;
			} else if (t < (2/2.75)) {
				return c*(7.5625*(t-=(1.5/2.75))*t + 0.75) + b;
			} else if (t < (2.5/2.75)) {
				return c*(7.5625*(t-=(2.25/2.75))*t + 0.9375) + b;
			}
			return c*(7.5625*(t-=(2.625/2.75))*t + 0.984375) + b;
		},      
		bounceBoth: function(t, b, c, d){
			if (t < d/2) {
				return Tween['bounceIn'](t*2, 0, c, d) * 0.5 + b;
			}
			return Tween['bounceOut'](t*2-d, 0, c, d) * 0.5 + c*0.5 + b;
		}
	};

### tap函数的封装

	/*
		el：要点击的元素
		fn：点击之后要执行的事件

		在点击的时候，记录手指坐标
		抬起的时候，判断手指坐标和按下的手指坐标的差值，小于一定值时我们就认定它是点击
	*/
	function tap(el, fn){
		var startPoint = {};

		el.addEventListener('touchstart', function(e){
			startPoint = {
				x: e.changedTouches[0].pageX,
				y: e.changedTouches[0].pageY
			}
		});

		el.addEventListener('touchend', function(e){
			var nowPoint = {
				x: e.changedTouches[0].pageX,
				y: e.changedTouches[0].pageY
			}
			
			if( Math.abs(nowPoint.x-startPoint.x)<5 && Math.abs(nowPoint.y-startPoint.y)<5 ){
				fn&&fn.call(el, e);
			}
		});
	}

----------

### css函数的封装

	function css(el, attr, val){
		var transformArrt = [
			'rotate',
			'rotateX',
			'rotateY',
			'rotateZ',
			'skewX',
			'skewY',
			'scale',
			'scaleX',
			'scaleY',
			'translateX',
			'translateY',
			'translateZ'
		]

		for(var i=0; i<transformArrt.length; i++){
			if( attr == transformArrt[i] ){	//如果attr等于transform中的一个值，就代表用户想要操作的是transform
				return transform(el, attr, val);
			}
		}

		if( val == undefined ){	// 当val不为空的时候，就是设置样式
			if( attr == 'opacity' ){
				el.style[attr] = val;
			}else{
				el.style[attr] = val+'px';
			}
		}else{		// 当val为空的时候，就是获取样式
			var val = getComputedStyle(el)[attr];
			val = parseFloat(val);
			return val?val:0;
		}
	}

	/* transform用来设置和获取transform相关的值 */
	function transform(el, attr, val){
		if( !el.transform ){
			el.transform = {
			}
		}

		/* 获取元素相应的值 */
		if( val == undefined ){
			return el.transform[attr]?el.transform[attr]:0;
		}

		el.transform[attr] = val;	// 记录对象中元素的值
		/*
			旋转 deg
			缩放
			斜切 deg
			位移 px
		*/
		var str = '';
		for( var s in el.transform ){
			switch(s){
    			case 'rotate':
    			case 'rotateX':
    			case 'rotateY':
    			case 'rotateZ':
    			case 'skewX':
    			case 'skewY':
    				str += s + '('+ el.transform[s] +'deg)';
    				break;
    			case 'scale':
    			case 'scaleX':
    			case 'scaleY':
    				str += s + '('+ el.transform[s] +')';
    				break;
    			case 'translateX':
    			case 'translateY':
    			case 'translateZ':
    				str += s + '('+ el.transform[s] +'px)';
    				break;
    		}
		}
    		
		// 我们所有的设置都存在transform这个对象中，在真正设置的时候，需要把transform的所有设置都连接过来
		el.style.WebkitTransform = el.style.transform = str;
	}

### 
	function startMove(init){
		var t = 0;
		var b = {};//样式的初始值
		var c = {};//样式的差值	
		var d = Math.ceil(init.time/16.7);
		cancelAnimationFrame(init.el.timer);
		for(var s in init.target) {
			b[s] = css(init.el,s);
			c[s] = init.target[s] - b[s];
		}
		init.el.timer = requestAnimationFrame(move);
		function move(){
			if(t > d || d == 0){
				cancelAnimationFrame(init.el.timer);
				init.callBack&&init.callBack.call(init.el);
			} else {
				t++;
				for(var s in init.target){
					var val = Tween[init.type](t,b[s],c[s],d);
					css(init.el,s,val);
				}
				init.callIn&&init.callIn.call(init.el);
				init.el.timer = requestAnimationFrame(move);
			}
		}
	}

### 幻灯片

	function swiper(init){
		var wrap = init.wrap;//滑屏元素的父级
		var scroll = wrap.children[0];//滑动的元素
		var startPonit = {};//手指的初始位置
		var startEl = {};// 元素的初始位置
		var dir = init.dir;
		var isFirst = true; //判断用户是否 是第一次执行move;
		var lastTime = 0;
		var lastTimeDis = 0;
		//var F = .4;//拉力
		var min = {
			x:0,
			y:0
		}
		var isDir = { //判断用户在对哪个方向进行滑屏
			x: false,
			y: false
		};
		var dirTranslate = {
			x: "translateX",
			y: "translateY"
		};
		var lastPoint = {};
		var lastDis = 0;//最后一次的手指移动距离
		css(scroll,"translateX",0);
		css(scroll,"translateY",0);
		css(scroll,"translateZ",0);
		scroll.style.minWidth = "100%";
		scroll.style.minHeight = "100%";
		wrap.addEventListener('touchstart', function(e) {
			cancelAnimationFrame(scroll.timer);
			init.start&&init.start.call(wrap,e);
			var touch = e.changedTouches[0];
			startPonit = {
				x: touch.pageX,
				y: touch.pageY
			};
			lastPoint = {
				x: touch.pageX,
				y: touch.pageY
			};
			startEl = {
				x: css(scroll,"translateX"),
				y: css(scroll,"translateY")
			};
			min = {
				x: wrap.clientWidth - scroll.offsetWidth,
				y: wrap.clientHeight - scroll.offsetHeight
			}
			lastTime = Date.now();
			lastTimeDis = 0;
			lastDis = 0;
		});
		wrap.addEventListener('touchmove', function(e) {
			var touch = e.changedTouches[0];
			var nowTime = Date.now();
			var nowPonit = {
				x: touch.pageX,
				y: touch.pageY
			};
			if(lastPoint.x == nowPonit.x
			&&lastPoint.y == nowPonit.y){
				return;
			}
			var dis = {
				x: nowPonit.x - startPonit.x,
				y: nowPonit.y - startPonit.y
			};
			if(isFirst){
				if(Math.abs(dis.x) > Math.abs(dis.y)) {
					isDir.x = true;
					isFirst = false;
				} else {
					isDir.y = true;
					isFirst = false;
				}
			 }
			var target = {
				x: startEl.x + dis.x,
				y: startEl.y + dis.y
			};
			if(isDir[dir]){ 
			 	if(init.backOut == "none"){//不允许用户滑动超出
			 		target[dir] = target[dir] > 0 ? 0:target[dir];
			 		target[dir] = target[dir] < min[dir] ? min[dir]:target[dir];
			 	} else if(init.backOut == "out"){ //超出添加回弹
			 		if(target[dir] > 0){
			 			//target[dir]的数值就是超出的距离
			 			var over = target[dir];
			 			var F = getF(over);
			 			over *= F;
			 			target[dir] = over;
			 		} else if(target[dir] < min[dir]){
			 			var over =  min[dir] - target[dir];
			 			var F = getF(over);
			 			over *= F;
			 			target[dir] = min[dir] - over;
			 		}
	
			 	}
			 	css(scroll,dirTranslate[dir],target[dir]);
				e.preventDefault();
				lastDis = nowPonit[dir] - lastPoint[dir];//获取最后一次的距离
				lastTimeDis = nowTime - lastTime;
			 }
			init.move&&init.move.call(wrap,e);
			lastPoint.x = nowPonit.x;
			lastPoint.y = nowPonit.y;
			lastTime = nowTime;
		});
		wrap.addEventListener('touchend', function(e) {
			var nowTime = Date.now();//手指抬起时的时间
			//console.log(lastDis,lastTimeDis);
			if(nowTime - lastTime >= 100){
				// 判断 当用户手指抬起时 和 最后一次移动的时候，有比较大的一个时间间隔，就可以认定 用户在抬起手指前有那么一段时间是按着不动的，那么我们也就不执行缓冲
				lastDis = 0;
			}
			var lastSpeed = lastDis/lastTimeDis;//最后一次移动速度
			//console.log(lastSpeed);
			lastSpeed = lastSpeed?lastSpeed:0;
			/* 最后一次的速度越大，位移出去的距离越大 */
			var s = lastSpeed*170;//缓冲出去的距离
	
			var now = css(scroll,dirTranslate[dir]);
			s = Math.round(s + now);
			if(s > 0){
				s = 0;
			} else if(s < min[dir]){
				s = min[dir];
			}
			if(dir == "x"){
				var target = {translateX: s}
			} else {
				var target = {translateY: s}
			}
			var time = Math.abs(s - now) * 1.15;
			startMove({
				el: scroll,
				type: "easeOutStrong",
				time: time,
				target: target,
				callIn: function(){
					init.move&&init.move.call(wrap,e);
				},
				callBack: function(){
					init.over&&init.over.call(wrap,e);
				}
			});
			isFirst = true;
			isDir = { 
				x: false,
				y: false
			};
			init.end&&init.end.call(wrap,e);
		});
		function getF(over){ 
			var max = dir == "x" ? wrap.clientWidth:wrap.clientHeight;
			var F = 1 - over/max;
			F = F < .4?.4:F;
			return F;
		}
	}

### 自定义滚动

	function swiper(init){
		var wrap = init.wrap;//滑屏元素的父级
		var scroll = wrap.children[0];//滑动的元素
		var startPonit = {};//手指的初始位置
		var startEl = {};// 元素的初始位置
		var dir = init.dir;
		var isFirst = true; //判断用户是否 是第一次执行move;
		var lastTime = 0;
		var lastTimeDis = 0;
		//var F = .4;//拉力
		var min = {
			x:0,
			y:0
		}
		var isDir = { //判断用户在对哪个方向进行滑屏
			x: false,
			y: false
		};
		var dirTranslate = {
			x: "translateX",
			y: "translateY"
		};
		var lastPoint = {};
		var lastDis = 0;//最后一次的手指移动距离
		css(scroll,"translateX",0);
		css(scroll,"translateY",0);
		css(scroll,"translateZ",0);
		scroll.style.minWidth = "100%";
		scroll.style.minHeight = "100%";
		wrap.addEventListener('touchstart', function(e) {
			cancelAnimationFrame(scroll.timer);
			init.start&&init.start.call(wrap,e);
			var touch = e.changedTouches[0];
			startPonit = {
				x: touch.pageX,
				y: touch.pageY
			};
			lastPoint = {
				x: touch.pageX,
				y: touch.pageY
			};
			startEl = {
				x: css(scroll,"translateX"),
				y: css(scroll,"translateY")
			};
			min = {
				x: wrap.clientWidth - scroll.offsetWidth,
				y: wrap.clientHeight - scroll.offsetHeight
			}
			lastTime = Date.now();
			lastTimeDis = 0;
			lastDis = 0;
		});
		wrap.addEventListener('touchmove', function(e) {
			var touch = e.changedTouches[0];
			var nowTime = Date.now();
			var nowPonit = {
				x: touch.pageX,
				y: touch.pageY
			};
			if(lastPoint.x == nowPonit.x
			&&lastPoint.y == nowPonit.y){
				return;
			}
			var dis = {
				x: nowPonit.x - startPonit.x,
				y: nowPonit.y - startPonit.y
			};
			if(isFirst){
				if(Math.abs(dis.x) > Math.abs(dis.y)) {
					isDir.x = true;
					isFirst = false;
				} else {
					isDir.y = true;
					isFirst = false;
				}
			 }
			var target = {
				x: startEl.x + dis.x,
				y: startEl.y + dis.y
			};
			if(isDir[dir]){ 
			 	if(init.backOut == "none"){//不允许用户滑动超出
			 		target[dir] = target[dir] > 0 ? 0:target[dir];
			 		target[dir] = target[dir] < min[dir] ? min[dir]:target[dir];
			 	} else if(init.backOut == "out"){ //超出添加回弹
			 		if(target[dir] > 0){
			 			//target[dir]的数值就是超出的距离
			 			var over = target[dir];
			 			var F = getF(over);
			 			over *= F;
			 			target[dir] = over;
			 		} else if(target[dir] < min[dir]){
			 			var over =  min[dir] - target[dir];
			 			var F = getF(over);
			 			over *= F;
			 			target[dir] = min[dir] - over;
			 		}
	
			 	}
			 	css(scroll,dirTranslate[dir],target[dir]);
				e.preventDefault();
				lastDis = nowPonit[dir] - lastPoint[dir];//获取最后一次的距离
				lastTimeDis = nowTime - lastTime;
			 }
			init.move&&init.move.call(wrap,e);
			lastPoint.x = nowPonit.x;
			lastPoint.y = nowPonit.y;
			lastTime = nowTime;
		});
		wrap.addEventListener('touchend', function(e) {
			var nowTime = Date.now();//手指抬起时的时间
			//console.log(lastDis,lastTimeDis);
			if(nowTime - lastTime >= 100){
				// 判断 当用户手指抬起时 和 最后一次移动的时候，有比较大的一个时间间隔，就可以认定 用户在抬起手指前有那么一段时间是按着不动的，那么我们也就不执行缓冲
				lastDis = 0;
			}
			var lastSpeed = lastDis/lastTimeDis;//最后一次移动速度
			//console.log(lastSpeed);
			lastSpeed = lastSpeed?lastSpeed:0;
			/* 最后一次的速度越大，位移出去的距离越大 */
			var s = lastSpeed*170;//缓冲出去的距离
	
			var now = css(scroll,dirTranslate[dir]);
			s = Math.round(s + now);
			if(s > 0){
				s = 0;
			} else if(s < min[dir]){
				s = min[dir];
			}
			if(dir == "x"){
				var target = {translateX: s}
			} else {
				var target = {translateY: s}
			}
			var time = Math.abs(s - now) * 1.15;
			startMove({
				el: scroll,
				type: "easeOutStrong",
				time: time,
				target: target,
				callIn: function(){
					init.move&&init.move.call(wrap,e);
				},
				callBack: function(){
					init.over&&init.over.call(wrap,e);
				}
			});
			isFirst = true;
			isDir = { 
				x: false,
				y: false
			};
			init.end&&init.end.call(wrap,e);
		});
		function getF(over){ 
			var max = dir == "x" ? wrap.clientWidth:wrap.clientHeight;
			var F = 1 - over/max;
			F = F < .4?.4:F;
			return F;
		}
	}

### 多指操作

	/*init:{
	    el: 要添加事件的元素,
	    start: 按下时要操作的事件gesturestart,
	    change:{ 手指移动时的回调gesturechange
	        e.scale     在change时，手指之间的距离和start时手指之间距离的比值,
	        e.rotation  在change时和start时，手指旋转角度的差值
	    },
	    end: 多指操作结束的回调
	}*/

	function gesture(init){
	    var isGesture = false;
	    var el = init.el;
	    var startDis = 0;
	    var startDeg = 0;
	    var startDeg = 0;
	
	    el.addEventListener('touchstart', function(e){
	        var touch = e.touches;
	        
	        if( touch.length >=2 ){ //当屏幕上有两根以上的手指
	            isGesture = true;
	            startDis = getDis(touch[0], touch[1]);
	            startDeg = getDeg(touch[0], touch[1]);
	            init.start&&init.start.call(el, e);
	        }
	    });
	
	    el.addEventListener('touchmove', function(e){
	        var touch = e.touches;
	        
	        if( touch.length >=2 && isGesture ){ //当屏幕上有两根以上的手指
	            isGesture = true;
	            var nowDis = getDis(touch[0], touch[1]);
	            var nowDeg = getDeg(touch[0], touch[1]);
	
	            e.scale = nowDis/startDis;
	            e.rotation = nowDeg - startDeg;
	            init.change&&init.change.call(el, e);
	        }
	    })
	
	    el.addEventListener('touchend', function(e){
	        if( isGesture ){
	            init.end&&init.end.call(el, e);
	        }
	        isGesture = false;
	    })
	
	    function getDis(point1, point2){
	        var x = point2.pageX - point1.pageX;
	        var y = point2.pageY - point1.pageY;
	
	        return Math.sqrt(x*x+y*y);
	    }
	
	    function getDeg(point1, point2){
	    	var x = point2.pageX - point1.pageX;
	        var y = point2.pageY - point1.pageY;
	
	        return Math.atan2(y, x)*180/Math.PI;
	    }
	}
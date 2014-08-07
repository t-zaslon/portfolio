Класс написан с использованием Mootols для слайдера, в котором могут находится как картинки, так и видео(YouTube) .
Реализованы настройки видео через API YouTube, такие как остановка видео при прокрутке слайдера, воспроизведение
просмотра видео с того места, где оно было остановлено.

Слайдер работает на сайте [http://events.owox.ua/conf/2014/](http://events.owox.ua/conf/2014/)

var SliderFade_class = new Class({Implements: Options,initialize: function(options) {
        if (options) {
            this.setOptions(options)
        }
        this._elements_parent = $(this.options.elements_parent);
        this._elements = [];
        this._parent = $(this.options.parent);
        this._controls = {};
        this._item = this.options.item;
        this._img_name = this.options.img_name;
        this._video_name = this.options.video_name;
        this._selected_class = this.options.selected_class;
        this._disable_class = this.options.disable_class;
        this._li_list = this._elements_parent.children;
        this._trigger = false;
        this._slide_time = 8000;
        this._reload_time = 6000;
        this._auto_scroll = this._reload_scroll = null;
        if (this._getElements().length > 1) {
            this.startAutoScroll()
        }
    },startAutoScroll: function() {
        this._auto_scroll = setInterval(function() {
            this._controlsClickHandler('right', false)
        }.bind(this), this._slide_time)
    },reloadAutoScroll: function() {
        clearTimeout(this._reload_scroll);
        this._reload_scroll = null;
        this._reload_scroll = setTimeout(function() {
            this.startAutoScroll()
        }.bind(this), this._reload_time)
    },run: function() {
        var left = this._getControl('left'), right = this._getControl('right');
        left.addEvent('click', function(e) {
            new DOMEvent(e).stop();
            this._controlsClickHandler('left', true)
        }.bind(this));
        right.addEvent('click', function(e) {
            new DOMEvent(e).stop();
            this._controlsClickHandler('right', true)
        }.bind(this));
        return this
    },_getElements: function() {
        var that = this, list = [];
        if (this._li_list) {
            $$(this._li_list).each(function(el) {
                var item = el.getElement("." + that._item), item_name = item.get('name');
                switch (item_name) {
                    case that._img_name:
                        list.include(new ItemImage_class(item));
                        break;
                    case that._video_name:
                        var current_embed = item.getElement('embed'), i = 0, video = new ItemVideo_class({element: item});
                        video._setPlayer(current_embed);
                        list.include(video);
                        break;
                    default:
                        break
                }
            })
        }
        return this._elements = list
    },_getControl: function(direction) {
        if (this._controls[direction] == null) {
            this._controls[direction] = this._parent.getElement(this.options.controls[direction])
        }
        return this._controls[direction]
    },_getSelected: function() {
        var that = this, selected;
        this._elements.each(function(el) {
            if (el.element.getParent().hasClass(that._selected_class)) {
                selected = el
            }
        });
        return selected
    },_getItem: function(parent_item) {
        return parent_item.getElement("." + this._item)
    },_controlsClickHandler: function(direction, stop_auto_scroll) {
        if (stop_auto_scroll) {
            if (this._auto_scroll) {
                clearInterval(this._auto_scroll);
                this._auto_scroll = null
            }
            App.trackGTMEvent({'eventCategory': 'Interactions','eventAction': 'Scroll','eventLabel': 'BigPromo'});
            this.reloadAutoScroll()
        }
        if (this._elements && this._elements.length) {
            var selected_item = this._getSelected(), player_state;
            if (selected_item instanceof ItemVideo_class) {
                player_state = selected_item._getPlayer().getPlayerState();
                if (player_state == 1 || player_state == 5) {
                    selected_item._getPlayer().pauseVideo()
                }
            }
            if (!this._trigger) {
                this._fadeItem(direction)
            }
        }
    },_fadeItem: function(direction) {
        this._trigger = true;
        var that = this, selected_Item = this._getSelected();
        var selected_parent_item = selected_Item.element.getParent();
        var next_Item = this._getSubling(direction);
        var next_parent_item = next_Item.element.getParent();
        selected_Item.hide();
        selected_parent_item.set('tween', {duration: 700}).fade('out');
        next_parent_item.set('tween', {duration: 800,onComplete: function() {
                selected_parent_item.removeClass(that._selected_class);
                next_parent_item.addClass(that._selected_class);
                that._trigger = false
            }}).fade('in');
        next_Item.show()
    },_getSubling: function(direction) {
        var selected_item = this._getSelected(), current_index = this._elements.indexOf(selected_item), next_item;
        if (selected_item) {
            switch (direction) {
                case 'left':
                    if ((current_index + 1) >= this._elements.length) {
                        next_item = this._elements[0]
                    } else {
                        next_item = this._elements[current_index + 1]
                    }
                    break;
                case 'right':
                    if ((current_index - 1) < 0) {
                        next_item = this._elements[this._elements.length - 1]
                    } else {
                        next_item = this._elements[current_index - 1]
                    }
                    break
            }
            return next_item
        }
    },stopAutoScroll: function() {
        if (this._auto_scroll) {
            clearInterval(this._auto_scroll);
            this._auto_scroll = null
        }
        if (this._reload_scroll) {
            clearInterval(this._reload_scroll);
            this._reload_scroll = null
        }
    }});
var Item_class = new Class({initialize: function(item_element) {
        this.element = item_element;
        this.duration_hide = 500;
        this.duration_show = 700
    },hide: function() {
        this.element.set("tween", {duration: this.duration_hide}).fade('out')
    },show: function() {
        this.element.set('tween', {duration: this.duration_show}).fade('in')
    }});
var ItemImage_class = new Class({Implements: Item_class,initialize: function(item_element) {
        this.element = item_element;
        this.duration_hide = 700;
        this.duration_show = 900
    }});
var ItemVideo_class = new Class({Implements: [Item_class, Options],initialize: function(options) {
        if (options) {
            this.setOptions(options)
        }
        this.element = this.options.element;
        this.duration_hide = 700;
        this.duration_show = 700;
        this.player
    },hide: function() {
        var current_embed = this.element.getElement("embed");
        current_embed.set("styles", {visibility: 'hidden'});
        this.element.set("tween", {duration: this.duration_hide}).fade('out')
    },show: function() {
        var current_embed = this.element.getElement("embed");
        this.element.set('tween', {duration: this.duration_show}).fade('in');
        current_embed.set('styles', {visibility: 'visible'})
    },_getPlayer: function() {
        return this.player
    },_setPlayer: function(player) {
        this.player = player
    }});
    
    
    
    /////////////////////////////////////////////////////////////////////////////////
    
    
    //Инициализация и запуск слайдера для BigPromo 
			bigPromoSlider = new SliderFade_class({
				parent: 'bigpromo',//id
				elements_parent: 'promo-content',//id
				elements:'.bp-list-i',//class
				controls: {left: '[name=control-left]', right: '[name=control-right]'},//names
				item: "bp-item",//class
				img_name: "bp-img",//name
				video_name: "bp-video",//name
				selected_class: 'bp-selected',//class
				disable_class: {left: 'bp-disable-al', right: 'bp-disable-ar'}//object
			});

			bigPromoSlider.run();
			//Функция-обработчик события  onStateChange
			function stopScroll(newState) {
				//Если видео воспроизводится, запускаем функцию остановки автопрокрутки
				if (newState == 1) {

					bigPromoSlider.stopAutoScroll();
				}
			}

			function onYouTubePlayerReady(playerid) {

				//Подписываем плеер на событие*
				var ytplayer = document.getElementById(playerid);
				ytplayer.addEventListener('onStateChange', 'stopScroll');
			}
    
    
    

# bitrix_backet
Всем, Примеры частных решений, Секретные разработки в 1С-Битрикс

Мой опыт работы с BX.PopupWindow
Доброго времени суток, уважаемое сообщество...
Сей пост про использование стандартной библиотеки всплывающих окон PopupWindow

Для начала тем, кто еще не знаком, стоит познакомиться с официальной документацией  и с постом:

Итак ...

Была у меня задача - сделать аяксовую сменую 3-х отображений списка товаров инфоблока. Это успешно было реализовано. Все обрадовались, попили пива ... А дальше начали детально все тестировать и обнаружили, что стандартная всплывашка, которая появилась в eshop и реализованная через JCCatalogSection после аякс-показа элемента - просто не отрабатывает.  Потратил часа 4  - так и не разобрался, что же нужно сделать, чтобы работала ... Посему, решил сделать свою всплывашку.


Создал отдельный js-файл, подключил его сразу в хедере (знаю, что можно было подключить  и только на тех страницах, где это нужно, но не стал уже заморачиваться.... хотя стоило бы:) ), а также подключаем в хедере popup:
```
CJSCore::Init(array("popup"));
```
Представляю вашему вниманию текст данного файла:

BX.ready(function(){
    var Confirmer = new BX.PopupWindow("add_basket-confirm", null, {
     content: '<div id="mainshadow"></div>'+'<h3>Товар успешно добавлен в корзину</h3>',
     closeIcon: {right: "20px", top: "10px"},
     titleBar: {content: BX.create("span", {html: 'Товар добавлен в корзину', 'props': {'className': 'access-title-bar'}})},
     zIndex: 0,
     offsetLeft: 0,
     offsetTop: 0,
     draggable: {restrict: false},
     overlay: {backgroundColor: 'black', opacity: '80' },  /* затемнение фона */
     buttons: [
      new BX.PopupWindowButton({
          text: "Перейти в корзину",
          className: "popup-window-button-accept",
          events: {click: function(){
           location.href="/personal/cart/";
          }}
      }),
      new BX.PopupWindowButton({
          text: "Продолжить покупки",
          className: "webform-button-link-cancel",
          events: {click: function(){
           this.popupWindow.close(); // закрытие окна
          }}
      })
     ]
    });

    function PAIAddToBasket(PRODUCT_ID,QntTy){
     $.post('/basketchange.php',
      {
          itemID: PRODUCT_ID,
          quantity:  QntTy
      },
      function(data){
          $('div#basket-line').html(data);
          Confirmer.show();
      }
     );
    }

    $('div.bx_content_section').on('click','.buy_btn',function(){
     var PRODUCT_ID = $(this).attr('product_id');
     var QntTy = 1;
     PAIAddToBasket(PRODUCT_ID,QntTy);
     return false;
    });

});

и в подвале помещаем див:

<div id="add_basket-confirm"></div>


ну и последний штрих: во всех видах отображения у ссылок, отвечающих за добавление товара в корзину, добавляем класс "buy_btn". Примерно вот так:
<a class="buy_btn" href="#" product_id="<?=$arItem['ID']?>" > В корзину </a>
обычно в атрибут "product_id" помещаю ID товара, который нужно добавить в корзину. Не совсем корректно, но ничего умнее пока не придумал:)

Суть скрипта: вешаем на ссылку с классом buy_btn обработчик, который получает ID товара и количество единиц, которое нужно добавить (в данном случае - всегда 1 :)  )и это все передаем в функцию, которая делает аякс- запрос к файлу, добавляющему товар в корзину (этот файл возвращает обратно компонент "Ссылка на корзину", который также обновляем при добавлении товара в корзину) и вызывает всплывашку.


Простите, если сбивчиво описал - у меня отпуск начался:) Всем, кто также как и я в отпуске - удачно отдохнуть!

P.S. Если не хотите портить дефолтные стили у шаблонов от eshop - просто удалите атрибут ID у ссылок  добавления в корзину:)

UPD ATE 201-09-22

Увидел еще одну штуку:  если нужно какое-то событие после показа popup-окна, то добавляется еще один параметр, описывающий popup:


events: {
    onAfterPopupShow: function () {
        BX.ajax.post(
            '<? echo $this->GetFolder(); ?>/popup.php',
            {
                lang: BX.message('LANGUAGE_ID'),
                site_id: BX.message('SITE_ID') || '',
                arParams: curSetParams
 },
            BX.delegate(function (result) {
                    this.setContent(result);
                    BX("CatalogSetConstructor_" + element_id).style.left = (window.innerWidth - BX("CatalogSetConstructor_" + element_id).offsetWidth) / 2 + "px";
                    var popupTop = document.body.scrollTop + (window.innerHeight - BX("CatalogSetConstructor_" + element_id).offsetHeight) / 2;
                    //BX("CatalogSetConstructor_" + element_id).style.top = popupTop > 0 ? popupTop + "px" : 0;
 },
                this)
        );
    }
} 
Это увидел в компоненте catalog.se t.constructor. Оставил так, как было в исходнике. Суть: сначала в попапе показывается прелоадер, а при открытии из файла popup.php тянется содержимое попапа ...  Но, конечно же, можно сделать еще кучу других очень полезных фишек, используя данный метод ...

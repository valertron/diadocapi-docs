Как отправить товарную накладную ТОРГ-12
========================================

Рассмотрим последовательность действий к функциям интеграторского интерфейса Диадока, которые требуется совершить при отправке товарной накладной ТОРГ-12 в рекомендованном ФНС формате.

#. Продавец формирует файл титула продавца, подписывает и направляет покупателю.

#. Покупатель получает товарную накладную, подписанную и отправленную продавцом;

#. Покупатель формирует файл титула покупателя, подписывает своей ЭП и отправляет в адрес продавца.


.. note:: Более подробно о порядке обмена электронными накладными между компаниями можно почитать на `сайте <http://www.diadoc.ru/docs/others/tn>`__

Формирование файла титула продавца
----------------------------------

Если на стороне интеграционного решения не предусмотрено функциональности для формирования XML-документов, соответствущих утвержденным форматам, то продавец может сгенерировать файл титула, используя команду :doc:`../http/GenerateTorg12XmlForSeller`.
	   
В теле запроса должны содержаться данные для изготовления титула продавца для товарной накладной ТОРГ-12 в XML-формате в виде сериализованной структуры :doc:`Torg12SellerTitleInfo <../proto/Torg12Info>`.
	   
HTTP-запрос для генерации файла титула продавца товарной накладной ТОРГ-12 выглядит следующим образом:

::

    POST /GenerateTorg12XmlForSeller HTTP/1.1
    Host: diadoc-api.kontur.ru
    Authorization: DiadocAuth ddauth_api_client_id=testClient-8ee1638deae84c86b8e2069955c2825a
    Content-Length: 1252
    Connection: Keep-Alive

    <Сериализованная структура Torg12SellerTitleInfo>

В теле ответа содержится XML-файл титула продавца, построенный на основании данных из запроса.

Успешный ответ сервера выглядит так:
::

    HTTP/1.1 200 OK
    Content-Length: 598

    <XML-файл титула продавца>

Файл генерируется в соответствии с `XML-схемой <https://diadoc.kontur.ru/sdk/xsd/DP_OTORG12_1_986_00_05_01_02.xsd>`__, которой должны удовлетворять XML-накладные, согласно приказу ФНС.

Имя файла титула продавца для товарной накладной возвращается в стандартном HTTP-заголовке ``Content-Disposition``.

Отправка файла титула продавца
------------------------------

После того, как у вас есть XML-файл титула продавца, его нужно отправить с помощью команды :doc:`../http/PostMessage`. 

Для этого нужно подготовить структуру :doc:`../proto/MessageToPost` следующим образом:

-  в значение атрибута *FromBoxId* указываем идентификатор ящика отправителя;

-  в значение атрибута *ToBoxId* указываем идентификатор ящика получателя;

-  для передачи XML-файла титула продавца товарной накладной ТОРГ-12 нужно использовать атрибут *XmlTorg12SellerTitles*, описываемый структурой *XmlDocumentAttachment*:

	-  внутри структуры *XmlDocumentAttachment* находится вложенная структура *SignedContent*;
	
	-  сам XML-файл нужно передать в атрибут *Content*, подпись продавца в атрибут *Signature*.
	   
Описание структур, используемых при отправке товарной накладной ТОРГ-12:

.. code-block:: protobuf

    message MessageToPost {
        required string FromBoxId = 1;
        optional string ToBoxId = 2;
        repeated XmlDocumentAttachment XmlTorg12SellerTitles = 3;
    }

    message XmlDocumentAttachment {
        required SignedContent SignedContent = 1;
    }

    message SignedContent {
        optional bytes Content = 1;
        optional bytes Signature = 2;
    }

После отправки в теле ответа будет содержаться отправленное сообщение, сериализованное в протобуфер :doc:`../proto/Message`.

Все дальнейшие действия происходят на стороне покупателя.

SDK
---

Пример кода на C# для отправки файла титула продавца для товарной накладной ТОРГ-12:

.. code-block:: csharp

	// формирование файла титула продавца
	private GeneratedFile GenerateTorg12SellerTitle()
	{
		var content = new Torg12SellerTitleInfo()
			{
				// заполняем согласно структуре Torg12SellerTitleInfo
			};
		return api.GenerateTorg12XmlForSeller(authToken, content);
	}
		
	// отправка файла титула продавца
	private void SendTorg12SellerTitle()
	{
		var fileToSend = GenerateTorg12SellerTitle();

		var messageAttachment = new XmlDocumentAttachment
		{
			SignedContent = new SignedContent //файл подписи
			{
				Content = fileToSend.Content,
				Signature = new byte[0] //подпись продавца
			}
		};

		var messageToPost = new MessageToPost
		{
			FromBoxId = "идентификатор ящика отправителя",
			ToBoxId = "идентификатор ящика получателя",
			XmlTorg12SellerTitles = { messageAttachment }
		};

		api.PostMessage(authToken, messageToPost); //см. "Как авторизоваться в системе"
	}

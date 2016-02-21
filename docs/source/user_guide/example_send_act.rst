Как отправить акт о выполнении работ/оказании услуг
===================================================

Рассмотрим последовательность действий к функциям интеграторского интерфейса Диадока, которые требуется совершить при отправке акта о выполнении работ/оказании услуг в рекомендованном ФНС формате.

#. Исполнитель формирует файл титула исполнителя, подписывает и направляет заказчику.

#. Заказчик получает акт, подписанный и отправленный исполнителем;

#. Заказчик формирует файл титула заказчика, подписывает своей ЭП и отправляет в адрес исполнителя.


.. note:: Более подробно о порядке обмена электронными актами о выполнении работ/оказании услуг между компаниями можно почитать на `сайте <http://www.diadoc.ru/docs/others/acts>`__

Формирование файла титула исполнителя для акта
----------------------------------------------

Если на стороне интеграционного решения не предусмотрено функциональности для формирования XML-документов, соответствущих утвержденным форматам, то исполнитель может сгенерировать файл титула, используя команду :doc:`../http/GenerateAcceptanceCertificateXmlForSeller`.
	   
В теле запроса должны содержаться данные для изготовления титула исполнителя для акта о выполнении работ/оказании услуг в XML-формате в виде сериализованной структуры :doc:`AcceptanceCertificateSellerTitleInfo <../proto/AcceptanceCertificateInfo>`.
	   
HTTP-запрос для генерации файла титула исполнителя акта о выполнении работ/оказании услуг выглядит следующим образом:

::

    POST /GenerateAcceptanceCertificateXmlForSeller HTTP/1.1
    Host: diadoc-api.kontur.ru
    Authorization: DiadocAuth ddauth_api_client_id=testClient-8ee1638deae84c86b8e2069955c2825a
    Content-Length: 1252
    Connection: Keep-Alive

    <Сериализованная структура AcceptanceCertificateSellerTitleInfo>

В теле ответа содержится XML-файл титула исполнителя, построенный на основании данных из запроса.

Успешный ответ сервера выглядит так:
::

    HTTP/1.1 200 OK
    Content-Length: 598

    <XML-файл титула исполнителя>

Файл генерируется в соответствии с `XML-схемой <https://diadoc.kontur.ru/sdk/xsd/DP_ZAKTPRM_1_990_00_05_01_02.xsd>`__, которой должны удовлетворять XML-акты, согласно приказу ФНС.

Имя файла титула исполнителя для акта о выполнении работ/оказании услуг возвращается в стандартном HTTP-заголовке ``Content-Disposition``.

Отправка файла титула исполнителя для акта
------------------------------------------

После того, как у вас есть XML-файл титула исполнителя, его нужно отправить с помощью команды :doc:`../http/PostMessage`. 

Для этого нужно подготовить структуру :doc:`../proto/MessageToPost` следующим образом:

-  в значение атрибута *FromBoxId* указываем идентификатор ящика отправителя;

-  в значение атрибута *ToBoxId* указываем идентификатор ящика получателя;

-  для передачи XML-файла титула исполнителя акта о выполнении работ/оказании услуг нужно использовать атрибут *XmlAcceptanceCertificateSellerTitles*, описываемый структурой *XmlDocumentAttachment*:

	-  внутри структуры *XmlDocumentAttachment* находится вложенная структура *SignedContent*;
	
	-  сам XML-файл нужно передать в атрибут *Content*, подпись исполнителя в атрибут *Signature*.
	   
Описание структур, используемых при отправке акта о выполнении работ/оказании услуг:

.. code-block:: protobuf

    message MessageToPost {
        required string FromBoxId = 1;
        optional string ToBoxId = 2;
        repeated XmlDocumentAttachment XmlAcceptanceCertificateSellerTitles = 3;
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

Пример кода на C# для отправки файла титула исполнителя для акта о выполнении работ/оказании услуг:

.. code-block:: csharp

	// формирование файла титула исполнителя
	private GeneratedFile GenerateAcceptanceCertificateSellerTitle()
	{
		var content = new AcceptanceCertificateSellerTitleInfo()
			{
				// заполняем согласно структуре AcceptanceCertificateSellerTitleInfo
			};
		return api.GenerateAcceptanceCertificateXmlForSeller(authToken, content);
	}
		
	// отправка файла титула исполнителя
	private void SendAcceptanceCertificateSellerTitle()
	{
		var fileToSend = GenerateAcceptanceCertificateSellerTitle();

		var messageAttachment = new XmlDocumentAttachment
		{
			SignedContent = new SignedContent //файл подписи
			{
				Content = fileToSend.Content,
				Signature = new byte[0] //подпись исполнителя
			}
		};

		var messageToPost = new MessageToPost
		{
			FromBoxId = "идентификатор ящика отправителя",
			ToBoxId = "идентификатор ящика получателя",
			XmlAcceptanceCertificateSellerTitles = { messageAttachment }
		};

		api.PostMessage(authToken, messageToPost); //см. "Как авторизоваться в системе"
	}

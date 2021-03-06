**Скачать плагин можно по ссылке:**

https://github.com/pflb/Jmeter.Plugin.CsvLogWriter/blob/master/ru.pflb.jmeter.listener.CsvLogWriter.jar?raw=true

Есть проект Jmeter-plugins.org, с **jp@gc - Flexible File Writer**:

- standard\src\kg\apc\jmeter\reporters:
  - FlexibleFileWriter.java
  - FlexibleFileWriterGui.java

Задача - сделать:

- вывод текстов ошибок;
- вывод в csv-формат;
- ротацию.


D:\Project\jmeter-plugins\standard\src\kg\apc\jmeter\reporters\\**FlexibleFileWriter.java**, ключевой метод тут ``public void sampleOccurred(SampleEvent evt)``:

	
	@Override
    public void sampleOccurred(SampleEvent evt) {
        if (fileChannel == null || !fileChannel.isOpen()) {
            if (log.isWarnEnabled()) {
                log.warn("File writer is closed! Maybe test has already been stopped");
            }
            return;
        }

        ByteBuffer buf = ByteBuffer.allocateDirect(writeBufferSize);
        for (int n = 0; n < compiledConsts.length; n++) {
            if (compiledConsts[n] != null) {
                //noinspection SynchronizeOnNonFinalField
                synchronized (compiledConsts) {
                    buf.put(compiledConsts[n].duplicate());
                }
            } else {
                if (!appendSampleResultField(buf, evt.getResult(), compiledFields[n])) {
                    appendSampleVariable(buf, evt, compiledVars[n]);
                }
            }
        }

        buf.flip();

        try {
            syncWrite(buf);
        } catch (IOException ex) {
            log.error("Problems writing to file", ex);
        }
    }


В методе ``sampleOccurred`` удобно реализовать обработку подразапросов.
Для получения текстов ошибок удобно выводить первые 200-300 символов ответа в лог, для запросов с кодом ответа == 500.

Ротацию логов можно реализовать в методе ``syncWrite``, выполняя ротацию по количеству записанных байт в один файл.


**Описание функционала плагина pflb@CsvLogWriter.**

В ходе работы был написан плагин pflb@CsvLogWriter для JMeter. Данный плагин позволяет записывать подробный лог-файл 
без предварительных настроек. 
  Лог-файл фиксирует следующие данные: 
- timeStamp - начало обработки, мс;  
- elapsed - продолжительность обработки, мс;
- label - наименование компонента JMeter; 
- responseCode - код ответа на запрос; 
- responseMessage - содержание ответа на запрос;	
- threadName - наименование катушки;	
- dataType - тип данных;
- success - статус выполнения запроса;	
- failureMessage - сообщение об ошибке;
- bytes - объем данных ответа сервера;	
- grpThreads - количество активных виртуальных пользователей текущей группы;	
- allThreads - общее количество активных виртуальных пользователей всех групп; 
- URL - ссылка;	
- Filename - наименования файла, в который записываются ответы;	
- Latency - время до получения первого ответа сервера;	
- Encoding - кодировка;	
- SampleCount - количество семплов;	
- ErrorCount - количество ошибок;	
- Hostname - наименование машины;	
- IdleTime - время простоя, мс;
- Connect - время, затраченное на установку соединения;	
- headersSize - размер заголовков;
- bodySize - размера тела;
- contentType - тип содержимого из заголовка ответа;
- endTime - время конца запроса;
- threadName_label - наименование треда и компонента JMeter;
- parent_threadName_label - наименование треда и компонента JMeter родителя;
- startTime - время начала запроса;
- stopTest - признак остановлен ли тест - кнопка Stop;
- stopTestNow - признак остановлен ли тест резко - кнопка Shutdown;
- stopThread - признак остановлен ли текущий поток;
- startNextThreadLoop - стартует ли повтор;
- isTransactionSampleEvent - признак того, что текущее событие является транзакцией (TransactionController);
- transactionLevel - уровень вложенности запроса;
- responseDataAsString - полное содержание ошибки в формате строки в случае ее возникновения;
- requestHeaders - заголовки запроса;
- responseData - полное содержание ошибки в случае ее возникновения;
- responseHeaders - заголовки ответа;
- <Имя_переменной> - переменные JMeter.



  К ключевым особенностям данного плагина можно отнести то, что он может фиксировать результаты работы дочерних подзапросов и записывать полный текст ошибки, 
при ее возникновении, в виде обычного текста, а не в XML-формате.

Пример результата работы плагина:

    timeStamp;elapsed;label;responseCode;responseMessage;threadName;dataType;success;failureMessage;bytes;grpThreads;allThreads;URL;Filename;Latency;Encoding;SampleCount;ErrorCount;Hostname;IdleTime;Connect;"responseData";"transactionLevel"
    1454498796125;6759;"Transaction Controller1";"200";"Number of samples in transaction : 11, number of failing samples : 0";"Thread Group 1-1";"";true;"";1289;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";0
    1454498796125;540;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498796126;540;"jp@gc - Dummy Sampler";"200";"OK";"Thread Group 1-1";"text";true;"";114;1;1;"null";"";4;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";2
    1454498796667;667;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498796668;667;"jp@gc - Dummy Sampler";"200";"OK";"Thread Group 1-1";"text";true;"";114;1;1;"null";"";9;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";2
    1454498797336;666;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498802041;853;"jp@gc - Error Sampler";"500";"Internal Server Error";"Thread Group 1-1";"text";true;"";149;1;1;"null";"";87;"ISO-8859-1";1;0;"aperevozchikova";9;0;"Error text."

Для запуска плагина необходимо заполнить поле Filename. 
Поле Filename содержит путь к файлу, в котором будет вестись фиксация результатов работы. Можно прописать директорию вручную, или выбрать файл используя кнопку Browse. 
В случае, когда указанный файл существует, создается новый файл с добавлением постфикса номера файла. Так же на форме расположены флажки. С помощью флажков можно манипулировать данными, фиксируемыми в логе. Флажок Additional parameters отвечает за фиксацию дополнительных параметров, Response data отвечает за фиксацию текста ошибок, User variables отвечает за фиксацию пользовательских переменных.

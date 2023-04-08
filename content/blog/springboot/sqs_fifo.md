---
title: 'AWS sqs fifo 사용하기'
date: 2023-04-08 01:47:00
category: 'springboot'
draft: false
---

# AWS의 SQS(simple queue service)
- AWS의 sqs에는 standard 버전과 fifo 버전 2가지가 있는데, fifo 버전을 사용했고 간단한 기록을 남기려고 한다.
- fifo 버전은 중복되지 않고 순서를 반드시 지켜야 할 때 사용하면 좋다.
- 그런데 나는 꼭 순서가 중요하지는 않지만 중복이 일어나서는 안되는 작업이 필요했다. 여러 서버에서 sqs에 접근을 해서 메세지를 가져갈 때 각각 중복, 누락이 있어서는 안되었기 때문이다.
	- 예를 들어 sqs 메세제에(1, 2, 3, 4, 5, 6 ...100) 이 있다고 가정할 때 A, B, C 3개의 서버가 동시에 접근을 한다고 가정하자. 그러면 동시에 1에 접근을 했을 때 A, B, C 중 1개의 서버만 1이라는 값을 가져가야 하고 다른 2개의 서버는 중복해서 1이라는 값을 얻으면 안된다는 것.

- 참고로 환경은 SpringBoot 버전 2.17, java11 


## sqs 생성 후 url 가져오기
- 우선 AWS에 sqs(대기열) 을 만들어서 url을 얻었다고 가정을 하자. (생성은 생략)
- 이 url은 application.yaml 에 저장을 해두고 가져오기로 한다. 
- application.yaml
```
aws:
	sqs:
		fifo:
			mywork:
				url: https://sqs.ap-northeast-2.amazonaws.com/(이하 생략) # url 자체는 aws에서 생성해줌
```

- 이 url 에다가 sqs을 등록하는 것

## sqs 등록하기
```java
public class AwsSqsRemote {

	@Autowired
	private AmazonSQS amazonSqs;
	
	@Value(value = "${aws.sqs.fifo.mywork.url}")
	private String awsQueueMyWorkHost;

	private final String MESSAGE_GROUP_ID = "my_work_fifo_group";

	public void enRollMyWorkFifoSqs(Long userIdx) {

		SendMessageRequest sendMessageRequest = new SendMessageRequest()
			.withQueueUrl(awsQueueMyWorkHost)
			.withMessageBody(String.valueOf(userIdx))
			.withMessageGroupId(MESSAGE_GROUP_ID)
			.withMessageDeduplicationId(String.valueOf(userIdx))
			;

        this.amazonSqs.sendMessage(sendMessageRequest);
	}
}
```
- .withQueueUrl
	- aws에서 sqs 대기열을 생성한 url
	- aws에 여러 sqs 대기열을 생성할 수도 있으므로 해당하는 Url을 잘 가져와야 한다.

- .withMessageBody(String.valueOf(userIdx))
	- 메시지에는 크게 속성값(attribute) 와 메시지바디(message body) 2곳에 데이터를 담을 수 있는 듯 하다.
		- 속성값(attribute) 는 마치 HTTP의 헤더값과 비슷한 느낌이고, 메시지 바디는 RequestBody와 같은 것.
	- String.valueOf
		- messageBody에는 반드시 String 값이 들어가야 하므로 형 변환
	- userIdx
		- 메시지 바디에 내가 원하는 메시지 값을 넣으면 되는데, 나는 특정 user에게 `특정한 작업을 반드시 중복X, 누락X 하도록 하고 싶었다.` 그래서 나는 대상(user) 선정이 중요했으므로 간단하게 userIdx 만 넣었다.
	
- .withMessageGroupId(MESSAGE_GROUP_ID)
	- fifo 에서는 groupId 값이 반드시 필요하다. 함수 내부로 들어가서 docs description을 살펴보면 아래 내용들이 있다.(부분발췌)
	```
	This parameter applies only to FIFO (first-in-first-out) queues.
	
	You must associate a non-empty MessageGroupId with a message. If you don't provide a MessageGroupId, the action fails.
	
	The tag that specifies that a message belongs to a specific message group. Messages that belong to the same message group are processed in a FIFO 
	
	MessageGroupId is required for FIFO queues. You can't use it for Standard queues.
	(..중략..)
	```
	- sqs에서의 fifo 에서는 반드시 강제하기 때문에 넣어야 하고, 넣게 되면 같은 groupId 를 갖고 있는 메시지들은 순서를 반드시 보장해준다.
	- 나는 사실 순서가 중요하지는 않았고 그룹을 따로 세분화할 필요도 없었으므로 같은 groupId 를 넣었다. 사실 이 값도 설정에서 값을 설정해도 될 듯하다.

- withMessageDeduplicationId(String.valueOf(userIdx))
	- fifo 에서는 역시 deduplicationId 가 필수값이다. 함수 내부로 들어가서 살펴보면 아래 내용들이 있다.(부분발췌)
	```
	This parameter applies only to FIFO (first-in-first-out) queues.
	The token used for deduplication of sent messages. If a message with a particular MessageDeduplicationId is sent successfully, any messages sent with the same MessageDeduplicationId are accepted successfully but aren't delivered during the 5-minute deduplication interval. 
	
	Every message must have a unique MessageDeduplicationId.

	If you send one message with ContentBasedDeduplication enabled and then another message with a MessageDeduplicationId that is the same as the one generated for the first MessageDeduplicationId, the two messages are treated as duplicates and only one copy of the message is delivered.
	(..중략..)
	```
	- 위의 내용을 보면, 각 메시지들의 중복금지를 보장해준다.
	- 그래서 uuid를 넣어도 되지만, 난 이미 기존에 userIdx 자체가 고유한 값이었으므로 그대로 deduplicationId로 넣었다.

- 이렇게 넣은 값들은 aws 의 해당하는 대기열에 들어가서, '메시지 전송 및 수신' 을 클릭하여 수신한 메시지 목록 & 값들을 확인할 수 있다.


## sqs 값 꺼내기 및 삭제하기
- sqs 에서 메시지를 받아오면 바로 삭제를 했다.
	- 의문) 그러면 서버 다중화일 경우에 메시지를 받자마자(1, 2, 3) 삭제를 하기 직전에 다른 서버가 그 메시지(1, 2, 3)를 받을 확률이 있지 않나? 즉 중복.
		- 답은 없다. 왜냐하면 특정 메시지를 받아가면 해당 메시지에는 특정 시간동안 접근 자체가 되지 않는다.

```java
public class AwsSqsRemoteService {

	@Autowired
	private AmazonSQS amazonSqs;
	
	@Value(value = "${aws.sqs.fifo.mywork.url}")
	private String awsQueueMyWorkHost;

	private final String MESSAGE_GROUP_ID = "my_work_fifo_group";

	// sqs 등록
	public void enRollMyWorkFifoSqs(Long userIdx) {
		
		(..중략..)
	}

	// sqs 메시지 가져오기
	public List<Message> getSqsMyWorkMessages() {

		ReceiveMessageRequest receiveMessageRequest = new ReceiveMessageRequest()
				.withQueueUrl(this.queueAwsSikdaeHost)
				.withMaxNumberOfMessages(10);

		return this.amazonSqs.receiveMessage(receiveMessageRequest).getMessages();
    }

	// sqs 삭제하기
	public void deleteBatchSqsMessage(List<Message> messageList) {

        List<DeleteMessageBatchRequestEntry> entries = new ArrayList<>();
        messageList.stream()
                .forEach(message -> {
                    DeleteMessageBatchRequestEntry deleteMessageBatchRequestEntry = new DeleteMessageBatchRequestEntry();
                    deleteMessageBatchRequestEntry.setId(message.getMessageId());
                    deleteMessageBatchRequestEntry.setReceiptHandle(message.getReceiptHandle());
                    entries.add(deleteMessageBatchRequestEntry);
                });

        this.amazonSqs.deleteMessageBatch(this.queueAwsSikdaeHost, entries);
    }
}
```
- getSqsMyWorkMessages()
	- .withQueueUrl(this.queueAwsSikdaeHost)
		- 아까전에 sqs 메시지를 등록한 host

	- .withMaxNumberOfMessages(10);
		- aws sqs 에서는 한번에 꺼낼 수 있는 메시지가 1(default) ~ 10개밖에 안된다.

- deleteBatchSqsMessage()
	- DeleteMessageBatchRequestEntry()
		- sqs 메시지를 하나씩 삭제하는 것이 아니라 배치로 삭제하기
		- 하나씩 삭제해도 되지만 어차피 10개씩 불러와서 다 삭제해야 하니 배치로 삭제했다.
	- deleteMessageBatchRequestEntry.setId(message.getMessageId());
		- 삭제할 때는 반드시 id 값이 필요한데, 들고온 메시지의 getMessageId()를 그대로 넣으면 된다.
	- deleteMessageBatchRequestEntry.setReceiptHandle(message.getReceiptHandle());
		- 필수값이고 message.getReceiptHandle() 값 그대로 입력.


## sqs 메시지 값 꺼내서 비즈니즈 로직 수행
```java

    public void runSqsBusiness() {

        Boolean isMessageEnd = true;
        while (isMessageEnd) {
            List<Message> sqsMyWorkMessages = this.awsSqsRemote.getSqsMyWorkMessages();

            if (sqsMyWorkMessages.size() == 0) {
				isMessageEnd = false;
                break;
            }

            List<Long> userIdxListFromSqs = sqsMyWorkMessages
                    .stream()
                    .map(message -> Long.valueOf(message.getBody()))
                    .collect(Collectors.toList());
            this.awsSqsRemote.deleteBatchSqsMessage(sqsMyWorkMessages);

            for (Long userIdx : userIdxListFromSqs) {
				// business logic (userIdx 를 통해서 한 사람에 대해서 수행해야 할 비즈니스 로직 수행)
				
            }
        }
    }
```

## attribute(속성 값)
- 위에서는 사용하지 않았지만 여러가지 속성값을 사용하려면 아래와 같이 사용하면 된다.
```java

	public void sendSqs(List<String> bodies, String host, Map<String, MessageAttributeValue> attributeValueMap) {
        List<SendMessageBatchRequestEntry> entries = new ArrayList<>();
        bodies.forEach(body -> {
            String id = UUID.randomUUID().toString().toUpperCase();	// key 값 필요
            SendMessageBatchRequestEntry entry = new SendMessageBatchRequestEntry(id, body);
            entry.withMessageAttributes(attributeValueMap);
            entries.add(entry);
        });

        SendMessageBatchRequest sendMessageBatchRequest = new SendMessageBatchRequest()	// batch 로 보냄
                .withQueueUrl(host)
                .withEntries(entries);

        SendMessageBatchResult result = this.amazonSqs.sendMessageBatch(sendMessageBatchRequest);
        if (!ObjectUtils.isEmpty(result.getFailed())) {
            log.error("sendSqsBatchFailed :: {}", result.getFailed());
        }
    }


    private Map<String, MessageAttributeValue> getAttributeMap(String itemId, String quantity) {

        MessageAttributeValue itemIdValue = new MessageAttributeValue();
        itemIdValue.setStringValue(itemId);
        itemIdValue.setDataType("String");

        MessageAttributeValue quantityValue = new MessageAttributeValue();
        quantityValue.setStringValue(quantity);
        quantityValue.setDataType("String");

        Map<String, MessageAttributeValue> map = new HashMap<>();
        map.put("itemId", itemIdValue);
        map.put("quantity", quantityValue);
        return map;
    }
``
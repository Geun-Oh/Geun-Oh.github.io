+++
title = 'Terraform Provider'
date = 2024-08-21T12:00:38+09:00
draft = true
+++

## Terraform Provider 인턴일지

저번 주 처음 출근을 시작한 뒤, 오늘까지 지속해서 Object Storage 관련 내용들을 살펴보고 있다.

특히 NCP의 경우 Object Storage가 S3 Compatible하게 제작되었기 때문에, 별도의 sdk를 사용하지 않고 S3 sdk를 끌어와서 사용한다.
단순하게 `access_key`, `secret_key`, `endpoint`만 설정해주고, 이외에는 별도의 설정 없이 기존 API를 그대로 사용할 수 있다.

특히 이 과정에서 schema를 맞추는 것이 매우 일이었는데, AWS와 NCP 두 곳에서 모두 동작하도록 설정해주는 것이 쉽지 않았다.
이에 Bucket과 같은 경우는 고작 `id`, `bucket_name` 정도를 설정해주는 것에서 마무리할 수밖에 없었다.

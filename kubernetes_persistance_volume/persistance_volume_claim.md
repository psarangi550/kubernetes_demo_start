# persistance volume claim in kubernetes 

- we have set a up a `VolumeMount` for the `mongodb POD` which will map the `/data/db folder of the mongodb POD container` to `which will be stored inside the file system of the host`

- one  of the `power of kubernetes is that` we can define the `system configuration` as `VolumeMount where we define the which of the container file system we want to map` we can map it to `different system` such as `HostPath if we want to use the local path of the host system for dev testing` or we can use the `Amazon EBS volume` with the config as `awsElasticBlockStore insted of HostPath`

- but in the `real world scenario` we don't have only one POD which need the `Peristance Volume` rather `multiple POD in 100s` requiring the `Persistance Volume` in that case

- if we define like before and then decided to migrate the persistance storage from one platform to another platform then we have to change that in every POD definition , which can become very tidious to perform

- 
# Ingress annotations
You can add kubernetes annotations to ingress and service objects to customize their behavior.

!!!note
    - Annotations applied to service have higher priority over annotations applied to ingress. `Location` column below indicates where that annotation can be applied to.
    - Annotation keys and values can only be strings. Advanced format are encoded as below:
        - integer: '42'
        - stringMap: k1=v1,k2=v2
        - stringList: s1,s2,s3
        - json: 'jsonContent'
!!!tip
    The annotation prefix can be changed using the `--annotations-prefix` command line argument, by default it's `alb.ingress.kubernetes.io`, as described in the table below.

## Annotations
|Name                       | Type |Default|Location|
|---------------------------|------|------|------|
|[alb.ingress.kubernetes.io/actions.${action-name}](#actions)|json|N/A|ingress|
|[alb.ingress.kubernetes.io/auth-idp-cognito](#auth-idp-cognito)|json|N/A|ingress,service|
|[alb.ingress.kubernetes.io/auth-idp-oidc](#auth-idp-oidc)|json|N/A|ingress,service|
|[alb.ingress.kubernetes.io/auth-on-unauthenticated-request](#auth-on-unauthenticated-request)|authenticate\|allow\|deny|authenticate|ingress,service|
|[alb.ingress.kubernetes.io/auth-scope](#auth-scope)|string|openid|ingress,service|
|[alb.ingress.kubernetes.io/auth-session-cookie](#auth-session-cookie)|string|AWSELBAuthSessionCookie|ingress,service|
|[alb.ingress.kubernetes.io/auth-session-timeout](#auth-session-timeout)|integer|'604800'|ingress,service|
|[alb.ingress.kubernetes.io/auth-type](#auth-type)|none\|oidc\|cognito|none|ingress,service|
|[alb.ingress.kubernetes.io/backend-protocol](#backend-protocol)|HTTP \| HTTPS|HTTP|ingress,service|
|[alb.ingress.kubernetes.io/certificate-arn](#certificate-arn)|stringList|N/A|ingress|
|[alb.ingress.kubernetes.io/healthcheck-interval-seconds](#healthcheck-interval-seconds)|integer|'15'|ingress,service|
|[alb.ingress.kubernetes.io/healthcheck-path](#healthcheck-path)|string|/|ingress,service|
|[alb.ingress.kubernetes.io/healthcheck-port](#healthcheck-port)|integer \| traffic-port|traffic-port|ingress,service|
|[alb.ingress.kubernetes.io/healthcheck-protocol](#healthcheck-protocol)|HTTP \| HTTPS|HTTP|ingress,service|
|[alb.ingress.kubernetes.io/healthcheck-timeout-seconds](#healthcheck-timeout-seconds)|integer|'5'|ingress,service|
|[alb.ingress.kubernetes.io/healthy-threshold-count](#healthy-threshold-count)|integer|'2'|ingress,service|
|[alb.ingress.kubernetes.io/inbound-cidrs](#inbound-cidrs)|stringList|0.0.0.0/0|ingress|
|[alb.ingress.kubernetes.io/ip-address-type](#ip-address-type)|ipv4 \| dualstack|ipv4|ingress|
|[alb.ingress.kubernetes.io/listen-ports](#listen-ports)|json|'[{"HTTP": 80}]' \| '[{"HTTPS": 443}]'|ingress|
|[alb.ingress.kubernetes.io/load-balancer-attributes](#load-balancer-attributes)|stringMap|N/A|ingress|
|[alb.ingress.kubernetes.io/namespace.${service-name}](#service-namespace)|string|N/A|ingress|
|[alb.ingress.kubernetes.io/scheme](#scheme)|internal \| internet-facing|internal|ingress|
|[alb.ingress.kubernetes.io/security-groups](#security-groups)|stringList|N/A|ingress|
|[alb.ingress.kubernetes.io/ssl-policy](#ssl-policy)|string|ELBSecurityPolicy-2016-08|ingress|
|[alb.ingress.kubernetes.io/subnets](#subnets)|stringList|N/A|ingress|
|[alb.ingress.kubernetes.io/success-codes](#success-codes)|string|'200'|ingress,service|
|[alb.ingress.kubernetes.io/tags](#tags)|stringMap|N/A|ingress|
|[alb.ingress.kubernetes.io/target-group-attributes](#target-group-attributes)|stringMap|N/A|ingress,service|
|[alb.ingress.kubernetes.io/target-type](#target-type)|instance \| ip|instance|ingress,service|
|[alb.ingress.kubernetes.io/unhealthy-threshold-count](#unhealthy-threshold-count)|integer|'2'|ingress,service|

## Traffic Listening
Traffic Listening can be controlled with following annotations:

- <a name="listen-ports">`alb.ingress.kubernetes.io/listen-ports`</a> specifies the ports that ALB used to listen on.

    !!!note ""
        defaults to `'[{"HTTP": 80}]'` or `'[{"HTTPS": 443}]'` depends on whether `certificate-arn` is specified.

    !!!example
        ```
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}, {"HTTP": 8080}, {"HTTPS": 8443}]'
        ```

- <a name="ip-address-type">`alb.ingress.kubernetes.io/ip-address-type`</a> specifies the [IP address type](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html#ip-address-type) of ALB.

    !!!example
        ```
        alb.ingress.kubernetes.io/ip-address-type: ipv4
        ```

## Traffic Routing
Traffic Routing can be controlled with following annotations:

- <a name="target-type">`alb.ingress.kubernetes.io/target-type`</a> specifies how to route traffic to pods. You can choose between `instance` and `ip`:

    - `instance` mode will route traffic to all ec2 instances within cluster on [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) opened for your service.

        !!!note ""
            service must be of type "NodePort" or "LoadBalancer" to use `instance` mode

    - `ip` mode will route traffic directly to the pod IP.

        !!!note ""
            network plugin must use secondary IP addresses on [ENI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) for pod IP to use `ip` mode. e.g.

            - [amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s)

    !!!example
        ```
        alb.ingress.kubernetes.io/target-type: instance
        ```

- <a name="backend-protocol">`alb.ingress.kubernetes.io/backend-protocol`</a> specifies the protocol used when route traffic to pods.

    !!!example
        ```
        alb.ingress.kubernetes.io/backend-protocol: HTTPS
        ```

- <a name="subnets">`alb.ingress.kubernetes.io/subnets`</a> specifies the [Availability Zone](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) that ALB will route traffic to. See [Load Balancer subnets](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-subnets.html) for more details.

    !!!note ""
        You must specify at least two subnets in different AZ. both subnetID or subnetName(Name tag on subnets) can be used.

    !!!tip
        You can enable subnet auto discovery to avoid specify this annotation on every ingress. See [Subnet Auto Discovery](../controller/config.md#subnet-auto-discovery) for instructions.

    !!!example
        ```
        alb.ingress.kubernetes.io/subnets: subnet-xxxx, mySubnet
        ```

- <a name="actions">`alb.ingress.kubernetes.io/actions.${action-name}`</a> Provides a method for configuring custom actions on a listener, such as for Redirect Actions.

    The `action-name` in the annotation must match the serviceName in the ingress rules, and servicePort must be `use-annotation`.

    !!!example
        - fixed 503 response
            ```yaml
            apiVersion: extensions/v1beta1
            kind: Ingress
            metadata:
              namespace: default
              name: ingress
              annotations:
                kubernetes.io/ingress.class: alb
                alb.ingress.kubernetes.io/actions.response-503: '{"Type": "fixed-response", "FixedResponseConfig": {"ContentType":"text/plain", "StatusCode":"503", "MessageBody":"503 error text"}}'
            spec:
              rules:
                - http:
                    paths:
                      - path: /503
                        backend:
                          serviceName: response-503
                          servicePort: use-annotation
            ```

- <a name="service-namespace">`alb.ingress.kubernetes.io/namespace.${service-name}`</a> Allows a LoadBalancer to target services in another namespace.

    The `service-name` in the annotation must match the serviceName in the ingress rules.

    !!!example
        - target service in demo namespace
            ```yaml
            apiVersion: extensions/v1beta1
            kind: Ingress
            metadata:
              namespace: default
              name: ingress
              annotations:
                kubernetes.io/ingress.class: alb
                alb.ingress.kubernetes.io/namespace.gameservice: 'demo'
            spec:
              rules:
                - http:
                    paths:
                      - path: /game
                        backend:
                          serviceName: gameservice
                          servicePort: http
            ```

## Access control
Access control for LoadBalancer can be controlled with following annotations:

- <a name="scheme">`alb.ingress.kubernetes.io/scheme`</a> specifies whether your LoadBalancer will be internet facing. See [Load balancer scheme](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#load-balancer-scheme) in the AWS documentation for more details.

    !!!example
        ```
        alb.ingress.kubernetes.io/scheme: internal
        ```

- <a name="inbound-cidrs">`alb.ingress.kubernetes.io/inbound-cidrs`</a> specifies the CIDRs that are allowed to access LoadBalancer.

    !!!warning ""
        this annotation will be ignored if `alb.ingress.kubernetes.io/security-groups` is specified.

    !!!example
        ```
        alb.ingress.kubernetes.io/inbound-cidrs: 10.0.0.0/24
        ```

- <a name="security-groups">`alb.ingress.kubernetes.io/security-groups`</a> specifies the securityGroups you want to attach to LoadBalancer.

    !!!note ""
        When this annotation is not present, the controller will automatically create 2 security groups: the first security group will be attached to the LoadBalancer and allow access from [`inbound-cidrs`](#inbound-cidrs) to the [`listen-ports`](#listen-ports). The second security group will be attached to the EC2 instance(s) and allow all TCP traffic from the first security group created for the LoadBalancer.

    !!!tip ""
        both name or ID of securityGroups are supported.

    !!!warning ""
        The [default limit](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_vpc) of security groups per network interface in AWS is 5. This limit is quickly reached when multiple load balancers are provisioned by the controller without this annotation, therefore it is recommended to set this annotation to a self-managed security group (or request AWS support to increase the number of security groups per network interface for your AWS account). If this annotation is specified, you should also manage the security group used by the EC2 instances to allow inbound traffic from the security group attached to the LoadBalancer.

    !!!example
        ```
        alb.ingress.kubernetes.io/security-groups: sg-xxxx, nameOfSg1, nameOfSg2
        ```

## Authentication
ALB supports authentication with Cognito or OIDC. See [Authenticate Users Using an Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html) for more details.

- <a name="auth-type">`alb.ingress.kubernetes.io/auth-type`</a> specifies the authentication type on targets.

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-type: cognito
        ```
        
- <a name="auth-idp-cognito">`alb.ingress.kubernetes.io/auth-idp-cognito`</a> specifies the cognito idp configuration.

    !!!tip ""
        If you are using Amazon Cognito Domain, the `UserPoolDomain` should be set to the domain prefix(xxx) instead of full domain(https://xxx.auth.us-west-2.amazoncognito.com)

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-idp-cognito: '{"UserPoolArn":"arn:aws:cognito-idp:us-west-2:xxx:userpool/xxx", "UserPoolClientId":"xxx", "UserPoolDomain":"xxx"}'
        ```

- <a name="auth-idp-oidc">`alb.ingress.kubernetes.io/auth-idp-oidc`</a> specifies the oidc idp configuration.
    
    !!!tip ""
        You need to create an [secret](https://kubernetes.io/docs/concepts/configuration/secret/) within the same namespace as ingress to hold your OIDC clientID and clientSecret. The format of secret is as below:
        ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
          namespace: testcase
          name: customizedSecretName
        data:
          clientId: base64 of your plain text clientId
          clientSecret: base64 of your plain text clientSecret
        ```

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-idp-oidc: '{"Issuer":"xxx","AuthorizationEndpoint":"xxx","TokenEndpoint":"xxx","UserInfoEndpoint":"xxx","SecretName":"customizedSecretName"}'
        ```

- <a name="auth-on-unauthenticated-request">`alb.ingress.kubernetes.io/auth-on-unauthenticated-request`</a> specifies the behavior if the user is not authenticated.
	
	!!!info "options:"
        * **authenticate**: try authenticate with configured IDP.
        * **deny**: return an HTTP 401 Unauthorized error.
        * **allow**: allow the request to be forwarded to the target.
    
    !!!example
        ```
        alb.ingress.kubernetes.io/auth-type: openid
        ```

- <a name="auth-scope">`alb.ingress.kubernetes.io/auth-scope`</a> specifies the set of user claims to be requested from the IDP(cognito or oidc).

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-type: openid
        ```

- <a name="auth-session-cookie">`alb.ingress.kubernetes.io/auth-session-cookie`</a> specifies the name of the cookie used to maintain session information

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-session-cookie: custom-cookie
        ```
        
- <a name="auth-session-timeout">`alb.ingress.kubernetes.io/auth-session-timeout`</a> specifies the maximum duration of the authentication session, in seconds

    !!!example
        ```
        alb.ingress.kubernetes.io/auth-session-timeout: '86400'
        ```

## Health Check
Health check on target groups can be controlled with following annotations:

- <a name="healthcheck-protocol">`alb.ingress.kubernetes.io/healthcheck-protocol`</a> specifies the protocol used when performing health check on targets.

    !!!tip ""
        default protocol can be set via `--backend-protocol` flag

    !!!example
        ```alb.ingress.kubernetes.io/healthcheck-protocol: HTTPS
        ```

- <a name="healthcheck-port">`alb.ingress.kubernetes.io/healthcheck-port`</a> specifies the port used when performing health check on targets.

    !!!example
        - set the healthcheck port to the traffic port
            ```
            alb.ingress.kubernetes.io/healthcheck-port: traffic-port
            ```
        - set the healthcheck port to the NodePort(when target-type=instance) or TargetPort(when target-type=ip) of a named port
            ```
            alb.ingress.kubernetes.io/healthcheck-port: my-port
            ```
        - set the healthcheck port to 80/tcp
            ```
            alb.ingress.kubernetes.io/healthcheck-port: '80'
            ```

    !!!warning ""
        When using `target-type: instance` with a service of type "NodePort", the healthcheck port can be set to `traffic-port` to automatically point to the correct port.

- <a name="healthcheck-path">`alb.ingress.kubernetes.io/healthcheck-path`</a> specifies the HTTP path when peforming health check on targets.

    !!!example
        ```
        alb.ingress.kubernetes.io/healthcheck-path: /ping
        ```

- <a name="healthcheck-interval-seconds">`alb.ingress.kubernetes.io/healthcheck-interval-seconds`</a> specifies the interval(in seconds) between health check of an individual target.

    !!!example
        ```
        alb.ingress.kubernetes.io/healthcheck-interval-seconds: '10'
        ```

- <a name="healthcheck-timeout-seconds">`alb.ingress.kubernetes.io/healthcheck-timeout-seconds`</a> specifies the timeout(in seconds) during which no response from a target means a failed health check

    !!!example
        ```
        alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '8'
        ```

- <a name="success-codes">`alb.ingress.kubernetes.io/success-codes`</a> specifies the HTTP status code that should be expected when doing health checks against the specified health check path.

    !!!example
        - use single value
            ```
            alb.ingress.kubernetes.io/success-codes: '200'
            ```
        - use multiple values
            ```
            alb.ingress.kubernetes.io/success-codes: 200,201
            ```
        - use range of value
            ```
            alb.ingress.kubernetes.io/success-codes: 200-300
            ```

- <a name="healthy-threshold-count">`alb.ingress.kubernetes.io/healthy-threshold-count`</a> specifies the consecutive health checks successes required before considering an unhealthy target healthy.

    !!!example
        ```
        alb.ingress.kubernetes.io/healthy-threshold-count: '2'
        ```

- <a name="unhealthy-threshold-count">`alb.ingress.kubernetes.io/unhealthy-threshold-count`</a> specifies the consecutive health check failures required before considering a target unhealthy.

    !!!example
        ```alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'
        ```

## SSL
SSL support can be controlled with following annotations:

- <a name="certificate-arn">`alb.ingress.kubernetes.io/certificate-arn`</a> specifies the ARN of one or more certificate managed by [AWS Certificate Manager](https://aws.amazon.com/certificate-manager)
    
    !!!tip ""
        The first certificate in the list will be added as default certificate. And remaining certificate will be added to the optional certificate list.
        See [SSL Certificates](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html#https-listener-certificates) for more details.
   
    !!!example
        - single certificate
            ```
            alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:xxxxx:certificate/xxxxxxx
            ```
        - multiple certificates
            ```
            alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:xxxxx:certificate/cert1,arn:aws:acm:us-west-2:xxxxx:certificate/cert2,arn:aws:acm:us-west-2:xxxxx:certificate/cert3
            ```

    !!!tip
        If the `alb.ingress.kubernetes.io/certificate-arn` annotation is not specified, the controller will attempt to add certificates to listeners that require it by matching available certs from ACM with the `host` field in each listener's ingress rule.

    !!!example
        - attaches a cert for `dev.example.com` or `*.example.com` to the ALB
            ```yaml
            apiVersion: extensions/v1beta1
            kind: Ingress
            metadata:
            namespace: default
            name: ingress
            annotations:
              kubernetes.io/ingress.class: alb
              alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
            spec:
            rules:
            - host: dev.example.com
              http:
                paths:
                - path: /users/*
                backend:
                  serviceName: user-service
                  servicePort: 80
            ```
    
    !!!tip
        Alternatively, domains specified using the `tls` field in the spec will also be matched with listeners and their certs will be attached from ACM. This can be used in conjunction with listener host field matching.
    
    !!!example
        - attaches certs for `www.example.com` to the ALB
            ```yaml
            apiVersion: extensions/v1beta1
            kind: Ingress
            metadata:
            namespace: default
            name: ingress
            annotations:
              kubernetes.io/ingress.class: alb
              alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
            spec:
              tls:
              - hosts:
                - www.example.com
            rules:
            - http:
                paths:
                - path: /users/*
                  backend:
                    serviceName: user-service
                    servicePort: 80
            ```
        
- <a name="ssl-policy">`alb.ingress.kubernetes.io/ssl-policy`</a> specifies the [Security Policy](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html#describe-ssl-policies) that should be assigned to the ALB, allowing you to control the protocol and ciphers.

    !!!example
        ```
        alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-1-2017-01
        ```

## Custom attributes
Custom attributes to LoadBalancers and TargetGroups can be controlled with following annotations:

- <a name="load-balancer-attributes">`alb.ingress.kubernetes.io/load-balancer-attributes`</a> specifies [Load Balancer Attributes](http://docs.aws.amazon.com/elasticloadbalancing/latest/APIReference/API_LoadBalancerAttribute.html) that should be applied to the ALB.

    !!!example
        - enable access log to s3
            ```
            alb.ingress.kubernetes.io/load-balancer-attributes:access_logs.s3.enabled=true,access_logs.s3.bucket=my-access-log-bucket,access_logs.s3.prefix=my-app
            ```
        - enable deletion protection
            ```
            alb.ingress.kubernetes.io/load-balancer-attributes:deletion_protection.enabled=true
            ```
        - enable http2 support
            ```
            alb.ingress.kubernetes.io/load-balancer-attributes:routing.http2.enabled=true
            ```

- <a name="target-group-attributes">`alb.ingress.kubernetes.io/target-group-attributes`</a> specifies [Target Group Attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#target-group-attributes) which should be applied to Target Groups.

    !!!example
        - set the slow start duration to 5 seconds
            ```
            alb.ingress.kubernetes.io/target-group-attributes: slow_start.duration_seconds=5
            ```
        - set the deregistration delay to 30 seconds
            ```
            alb.ingress.kubernetes.io/target-group-attributes: deregistration_delay.timeout_seconds=30
            ```
        - enable sticky sessions
            ```
            alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true,stickiness.lb_cookie.duration_seconds=60
            ```

## Resource Tags
ALB Ingress controller will automatically apply following tags to AWS resources(ALB/TargetGroups/SecurityGroups) created.

- `kubernetes.io/cluster/${cluster-name}:owned`
- `kubernetes.io/namespace: ${namespace}`
- `kubernetes.io/ingress-name: ${ingress-name}`

In addition, you can use annotations to specify additional tags

- <a name="tags">`alb.ingress.kubernetes.io/tags`</a> specifies additional tags that will be applied to AWS resources created.

    !!!example
        ```
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=test
        ```

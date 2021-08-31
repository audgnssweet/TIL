<h3> swagger3 사용하기 </h3>

    swagger 라는 API문서 자동화의 장점 때문에 swagger를 선호한다.
    3버전은 디자인이 예쁘다.

---

    1. gradle에 아래 설정을 추가해준다.
    
    implementation("io.springfox:springfox-boot-starter:3.0.0")

    2. 설정파일을 추가해준다.

    @Configuration
    @EnableOpenApi
    class SwaggerConfig {

        @Bean
        fun api(): Docket {
            return Docket(DocumentationType.OAS_30)
                .useDefaultResponseMessages(false)
                .groupName("V1")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(metaData())
        }
    
        private fun metaData(): ApiInfo {
            return ApiInfoBuilder()
                .title("EXAM REST API")
                .description("EXAM API V1")
                .version("V1")
                .build()
        }
    }

---

![image](https://user-images.githubusercontent.com/19279163/131513891-08d38db9-903e-4f25-b06d-6973738e55af.png)

성공!
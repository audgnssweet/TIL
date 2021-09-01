<h3> spring의 web request의 단발성에 관하여 </h3>

    이번에 프로젝트를 진행하면서 server error bot을 만드는 과정에서
    인상깊었던 점이 하나 있었다. 

    내가 원하는 정보를 Logback이 갖고있다가 중간에 오류가 생겼을 시 해당 에러를 slack으로 내보내기 위해서
    Filter단에서 미리 request를 가공했는데, 그냥 request를 읽어서 저장하는 방식이 아니라

    request의 inputStream을 읽어서 cachedStream에 저장하고 요청이 있을 때마다 cachedStream을
    return 하는 방식을 사용했다.

    java에서 원래 Stream은 단발성이라 이미 읽은 부분을 되돌려 다시 읽을 수가 없다.
    그런데 spring에서 web request도 body등을 inputStream으로 제공하기 때문에
    
    Filter단 (앞단) 에서 stream을 읽을 시 controller같은 곳에서 데이터가 이미 소멸되어 읽을 수 없는
    상태가 되는 것을 방지하기 위해서 취했던 방식이었던 것이다.

    그리하여 filter에서 request를 읽어 캐싱을 한다면, 몇번이고 request를 사용할 수 있는 것이다.

---

아래에 request를 caching하는 코드를 첨부하겠다.

```kotlin

class MultiReadableHttpServletRequest(request: HttpServletRequest) : HttpServletRequestWrapper(request) {

    private var cachedBytes: ByteArrayOutputStream? = null

    @Throws(IOException::class)
    override fun getInputStream(): ServletInputStream {
        if (cachedBytes == null) {
            cacheInputStream()
        }
        return CachedServletInputStream()
    }

    @Throws(IOException::class)
    private fun cacheInputStream() {
        cachedBytes = ByteArrayOutputStream()
        IOUtils.copy(super.getInputStream(), cachedBytes)
    }

    inner class CachedServletInputStream(
        private val input: ByteArrayInputStream = ByteArrayInputStream(cachedBytes!!.toByteArray())
    ) : ServletInputStream() {

        @Throws(IOException::class)
        override fun read() = input.read()
        override fun isFinished() = false
        override fun isReady() = true
        override fun setReadListener(listener: ReadListener?) {}
    }
}

```

---

    위의 코드에서 보면 request에 대해서 getInputStream() 메서드를 요청했을 경우에 
    캐싱이 되어있지 않다면 캐싱을 한 결과값을 주는 것을 볼 수 있다.
    
    cachedBytes에 원래 request의 inputStream을 IOUtils 를 통해 복사하는 것을 확인할 수 있다.

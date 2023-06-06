Filter that sets/deletes the traceid on MDC (mapped  diagnostic context)

```
@Component
@Order(1)
class SvvguidMDCFilter : OncePerRequestFilter() {

    internal companion object {
        const val SVVGUID = "Svvguid"
    }

    override fun doFilterInternal(request: HttpServletRequest, response: HttpServletResponse, chain: FilterChain) {
        try {
            request.getHeader(SVVGUID).let {
                val correlationId = if (it.isNullOrEmpty()) "uaginfo-${UUID.randomUUID()}" else it
                MDC.put(SVVGUID, correlationId)
            }
            chain.doFilter(request, response)
        } finally {
            MDC.remove(SVVGUID)
        }
    }
}
```

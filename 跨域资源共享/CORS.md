---
title: CORS 
tags: CORS
grammar_cjkRuby: true
---


``` stylus
public class SimpleCORSFilter implements Filter {

    protected static final Logger _log = LoggerFactory.getLogger(SimpleCORSFilter.class);

	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		_log.info("SimpleCORSFilter  过滤了!!!!!");
		HttpServletResponse response = (HttpServletResponse) res;
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
		response.setHeader("Access-Control-Max-Age", "3600");
		response.setHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept,token");
		response.setHeader("Access-Control-Expose-Headers", "token");
//		response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
		chain.doFilter(req, res);
		_log.info("SimpleCORSFilter end!!!!!");
	}

	public void init(FilterConfig filterConfig) {
	}

	public void destroy() {
	}
}
```

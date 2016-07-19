---
title: spring异常处理
date: 2016-07-12 12:49:02
categories: java
tags: spring
---
Spring 统一异常处理有 3 种方式，分别为：
1. 使用 @ ExceptionHandler 注解
1. 实现 HandlerExceptionResolver 接口
1. 使用 @ControllerAdvice 注解

<!--more-->

# 使用 @ ExceptionHandler 注解
使用该注解有一个不好的地方就是：进行异常处理的方法必须与出错的方法在同一个Controller里面,可以看到，这种方式最大的缺陷就是不能全局控制异常。每个类都要写一遍
```
@Controller     
public class GlobalController {              
 
   /**   
     * 用于处理异常的   
     * @return   
     */     
    @ExceptionHandler({MyException.class})      
    public String exception(MyException e) {      
        System.out.println(e.getMessage());      
        e.printStackTrace();      
        return "exception";      
    }      
 
    @RequestMapping("test")      
    public void test() {      
        throw new MyException("出错了！");      
    }                   
}    
```

#  实现 HandlerExceptionResolver 接口
这种方式可以进行全局的异常控制
```
@Component 
public class ExceptionTest implements HandlerExceptionResolver{ 
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, 
            Exception ex) { 
        System.out.println("This is exception handler method!"); 
        return null; 
    } 
}  
```

# 使用 @ControllerAdvice 注解
上文说到 @ ExceptionHandler 需要 进行异常处理的方法必须与出错的方法在同一个Controller里面。 那么当代码加入了 @ControllerAdvice，则不需要必须在同一个 controller 中了。这也是 Spring 3.2 带来的新特性。 也就是说，@ControllerAdvice + @ ExceptionHandler 也可以实现全局的异常捕捉。
那么，在实际中，就可以使用 @ControllerAdvice + @ ExceptionHandler，继承 ResponseEntityExceptionHandler 类来实现针对 Rest 接口 的全局异常捕获，并且可以返回自定义格式
```
@Slf4j
@ControllerAdvice
public class ExceptionHandlerBean  extends ResponseEntityExceptionHandler {
 
    /**
     * 数据找不到异常
     * @param ex
     * @param request
     * @return
     * @throws IOException
     */
    @ExceptionHandler({DataNotFoundException.class})
    public ResponseEntity<Object> handleDataNotFoundException(RuntimeException ex, WebRequest request) throws IOException {
        return getResponseEntity(ex,request,ReturnStatusCode.DataNotFoundException);
    }
 
    /**
     * 根据各种异常构建 ResponseEntity 实体. 服务于以上各种异常
     * @param ex
     * @param request
     * @param specificException
     * @return
     */
    private ResponseEntity<Object> getResponseEntity(RuntimeException ex, WebRequest request, ReturnStatusCode specificException) {
 
        ReturnTemplate returnTemplate = new ReturnTemplate();
        returnTemplate.setStatusCode(specificException);
        returnTemplate.setErrorMsg(ex.getMessage());
 
        return handleExceptionInternal(ex, returnTemplate,
                new HttpHeaders(), HttpStatus.OK, request);
    }
 
}
```
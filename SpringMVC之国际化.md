##һ.�������������Ĺ��ʻ�ʵ��

###1.������applicationContext.xml�ļ���ӵ���������
```
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
    <!-- ���ʻ���Ϣ���ڵ��ļ��� -->                     
    <property name="basename" value="messages" />   
    <!-- ����ڹ��ʻ���Դ�ļ����Ҳ�����Ӧ�������Ϣ���������������Ϊ����  -->               
    <property name="useCodeAsDefaultMessage" value="true" /> 
	<!-- �����ʽ -->
    <property name="defaultEncoding" value="UTF-8"/>	
</bean>
```
###2.����controller����ѡ��
```
@Controller
public class I18nController {

	@Autowired
	private LocaleResolver localeResolver;
	@RequestMapping(value = "ChangeI18n/{i18nType}")
	public String ChangeI18n(@PathVariable("i18nType") String i18nType, HttpServletRequest req, HttpServletResponse resp) throws Exception {

		Locale currentLocale = null;
		try {
			if (i18nType.equals("cn")) {
				currentLocale = new Locale("zh", "CN");
			} else if (i18nType.equals("us")) {
				currentLocale = new Locale("en", "US");
			}
			localeResolver.setLocale(req, resp, currentLocale);
		
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			return "/index";
		}

	}
}
```
3.index.jsp����ѡ������ѡ��
```
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%@ taglib uri="http://www.springframework.org/tags"  prefix="spring" %>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
  </head>
  <body>
  
      <a href="${pageContext.request.contextPath}/ChangeI18n/cn.do">����</a>  
      <a href="${pageContext.request.contextPath}/ChangeI18n/us.do">Ӣ��</a>       <br> <br> <br> <br> 
      
      <spring:message code="welcome"/>
      
      <a href="${pageContext.request.contextPath}/in18n.do" target="_blank">���ʻ�</a>
  </body>
</html>
```
##��.����Session�Ĺ��ʻ�ʵ��
###1.��applicationContext.xml�ļ���ӵ���������
```
<mvc:interceptors>  
    <!-- ���ʻ����������� ������û��ڣ�����/Session/Cookie����������� --> 
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />  
</mvc:interceptors>  

<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
```
###��.����Cookie�Ĺ��ʻ�ʵ�֣�
��ʵ�ֵڶ��ַ���ʱ����Ŀ��applicationContext.xml�ļ�����ӵ�
```
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
ע�͵���������������ݣ�

<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver" />
```
����<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver" />3�����Ե�˵��(���Զ������ö�����Ĭ��ֵ)
```
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">
    <!-- ����cookieName���ƣ����Ը�������ͨ��js���޸����ã�Ҳ������������ʾ�������޸����ã�Ĭ�ϵ�����Ϊ ����+LOCALE������org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE-->
    <property name="cookieName" value="lang"/>
    <!-- ���������Чʱ�䣬�����-1���򲻴洢��������رպ�ʧЧ��Ĭ��ΪInteger.MAX_INT-->
    <property name="cookieMaxAge" value="100000">
    <!-- ����cookie�ɼ��ĵ�ַ��Ĭ���ǡ�/��������վ���е�ַ���ǿɼ��ģ������Ϊ������ַ����ֻ�иõ�ַ�����ĵ�ַ�ſɼ�-->
    <property name="cookiePath" value="/">
</bean>
```
��.����URL����Ĺ��ʻ���ʵ��
###�������һ���࣬��������
```
import java.util.Locale;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.DispatcherServlet;
import org.springframework.web.servlet.LocaleResolver;

public class MyAcceptHeaderLocaleResolver extends AcceptHeaderLocaleResolver {

    private Locale myLocal;

    public Locale resolveLocale(HttpServletRequest request) {
        return myLocal;
    } 

    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
        myLocal = locale;
    }
  
}
```

Ȼ���ʵ�ֵڶ��ַ���ʱ����Ŀ��applicationContext.xml�ļ�����ӵ�
```
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
```

ע�͵���������������ݣ�

```
<bean id="localeResolver" class="xx.xxx.xxx.MyAcceptHeaderLocaleResolver"/>
```
��xx.xxx.xxx���Ǹղ���ӵ�MyAcceptHeaderLocaleResolver �����ڵİ�����

����������ԣ��޸�Ĭ�����Կ�������
```
/**
 * ���ʻ�������֤������Ϣ
 *
 * @param result
 * @param errorVoList
 */
public static void analyzeErrors(BindingResult result, ResponseEntity response, HttpServletRequest request) {
	// ��ȡ������֤������Ϣ
	List<ObjectError> errorList = result.getAllErrors();
	if (!errorList.isEmpty()) {
		// ������װ
		String message = errorList.get(0).getDefaultMessage();
		// �Ӻ�̨�����ȡ���ʻ���Ϣ
		RequestContext requestContext = new RequestContext(request);
		//��ù��ʻ�����
		Locale locate=requestContext.getLocale();
		//�ж������л�
		switch (locate.getLanguage()) {
			case "zh1": break;
			//����Ĭ������
			default:
				locate.setDefault(Locale.US);
				break;
		}
		response.setRtMessage(requestContext.getMessage(message));

		response.setRtCode(MsgConstants.PARAM_ERROR_CODE);
	}
}
```

����֮��Ϳ�����������Ϳ���ģ���ˣ����Ը����������������

����������ο�<a href="http://www.cnblogs.com/liukemng/p/3750117.html">����</a>





















# MVC

  ## 1.       MVC lifecycle
  
  ### MVC has two life cycles −
  
  1. The application life cycle
  
     Application life cycle is the time when the application process actually begins running on IIS until it get stop. It mark by the
     methods Application_Start() to Application_End() of Global.ashx.
  
  2. The request life cycle
  
  ## 2.       Routing
  ## 3.       Views
  ## 4.       Razor Views
  ## 5.       Filters
  ## 6.       Authentication
  ## 7.       Security implementation
  ## 8.       Exception handling
             
  ### 1. try/catch
             
                  public ActionResult Index()
                  {
                      try
                      {
                          int a = 1;
                          int b = 0;
                          int c = 0;
                          c = a / b; //it would cause exception. 

                          return View();
                      }
                      catch
                      {
                          Trace.Write("Error");

                          return View("Error");
                      }
                  }
                  
  ### 2. In web.config
                  
                  <system.web>
                    <customErrors mode="On" defaultRedirect="~/ErrorHandler/Index">
                        <error statusCode="404" redirect="~/ErrorHandler/NotFound"/>
                    </customErrors>
                  </system.web>
                               
  ### 3. using [HandleError] attribute
             
                  [HandleError(ExceptionType = typeof(DivideByZeroException), View = "~/Views/CommonExceptionView.cshtml")]
                  public ActionResult Contact()
                  {
                      int a = 1;
                      int b = 0;
                      int c = 0;
                      c = a / b; //it would cause exception.    
                      return View();
                  }
             
  ### 4. Overriding OnException() method of controller base class
             
                  public class HomeController : Controller
                  {
                      public ActionResult Index()
                      {
                          return View();
                      }

                      protected override void OnException(ExceptionContext filterContext)
                      {
                          filterContext.ExceptionHandled = true;

                          //Log the error!!
                          Trace.Write(filterContext.Exception);

                          //Redirect or return a view, but not both.
                          filterContext.Result = RedirectToAction("Index", "ErrorHandler");
                          // OR 
                          filterContext.Result = new ViewResult
                          {
                              ViewName = "~/Views/ErrorHandler/Index.cshtml"
                          };
                      }
                  }
                  
  ### 5. HandleErrorAttribute by create action filter class
                  
                  [CustomErrorHandling]
                  public ActionResult About()
                  {
                      int a = 1;
                      int b = 0;
                      int c = 0;
                      c = a / b; //it would cause exception.    

                      ViewBag.Message = "Your application description page.";

                      return View();
                  }
                  
                  public class CustomErrorHandling : HandleErrorAttribute
                  {
                      public override void OnException(ExceptionContext filterContext)
                      {
                          if (filterContext.ExceptionHandled || filterContext.HttpContext.IsCustomErrorEnabled)
                          {
                              return;
                          }
                          Exception e = filterContext.Exception;
                          filterContext.ExceptionHandled = true;
                          filterContext.Result = new ViewResult()
                          {
                              ViewName = "~/Views/CommonExceptionView.cshtml"
                          };
                      }

                  }

             
  ### 6. Application_Error() using Global.asax page.
                  
                  protected void Application_Error()
                  {
                      var ex = Server.GetLastError();
                      //log the error!
                      Trace.Write(ex);
                  }
  
  ## 9.       Caching
  ## 10.       Validations
  ## 11.       Areas
  ## 12.       Cookies
  ## 13.       Value Provider / Custom Value Provider
  ## 14.       handler
  
   Create a class library project and impliment "IHttpHandler"    
            public class RssHandler : IHttpHandler
            {
                public bool IsReusable
                {
                    get
                    {
                        return false;
                    }
                }

                public void ProcessRequest(HttpContext context)
                {
                    context.Response.ContentType = "text/html";

                    using (XmlWriter writer = XmlWriter.Create(context.Response.OutputStream))
                    {
                        writer.WriteStartDocument();
                        writer.WriteElementString("rss", "This is test feed.");
                        writer.WriteEndDocument();
                        writer.Flush();
                    }
                }
            } 
            
 Add Class library project reference in MVC project. Also add the below line in web.config            
                
                <system.webServer>
                  <handlers>
                    <add name="RssHandler" verb="*" path="*.rss" type="CustomHttpHandler.RssHandler, CustomHttpHandler"/>
                  </handlers>
                </system.webServer>
  
  ## 15.       module
              
    Create a class library project and impliment "IHttpModule"            
              
              public class LogginHttpModule : IHttpModule
              {
                  public void Dispose()
                  {

                  }

                  public void Init(HttpApplication context)
                  {
                      context.LogRequest += Context_LogRequest;
                  }

                  private void Context_LogRequest(object sender, EventArgs e)
                  {
                      HttpApplication application = (HttpApplication)sender;
                      HttpContext context = application.Context;

                      string requestPath = context.Request.Path;

                      Trace.WriteLine(String.Format("Request Path: {0}",requestPath));

                  }
              } 
              
      Add Class library project reference in MVC project.
      Also add the below line in web.config        
              
              <system.webServer>
                <modules>
                  <add name="LogginHttpModule" type="CustomHttpModule.LogginHttpModule, CustomHttpModule"/>
                </modules>
              </system.webServer>
              
  ## 16. PreApplicationStartMethod          
  
  This method will call before the Application_Start() method.
  
        [assembly: PreApplicationStartMethod(typeof(PreApplicationStartup), "Initialize")]
        namespace PreApplicationStartMethodDemo
        {
            public class PreApplicationStartup
            {

                public static void Initialize()
                {

                }
            }
        }
        
  or in side the "AssemblyInfo.cs"
  
    [assembly: PreApplicationStartMethod(typeof(PreApplicationStartup), "Initialize")]
  
  Now add the project reference in MVC project

 ## 17. ActionFilterAttribute
 
 Add a file with the name "CustomFilter" in the MVC project
 
    public class CustomFilter : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            string controllerName =filterContext.ActionDescriptor.ControllerDescriptor.ControllerName;
            string actionName = filterContext.ActionDescriptor.ActionName;
            Trace.Write(string.Format("OnActionExecuting : {0}, {1}", controllerName, actionName));
        }

        public override void OnActionExecuted(ActionExecutedContext filterContext)
        {
            string controllerName = filterContext.ActionDescriptor.ControllerDescriptor.ControllerName;
            string actionName = filterContext.ActionDescriptor.ActionName;
            Trace.Write(string.Format("OnActionExecuted : {0}, {1}", controllerName, actionName));
        }
    }
    
    [CustomFilter]
    public class HomeController : Controller
    {
    
    }

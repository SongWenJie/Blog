# Spring 注解

| **注解** | 解释 |
| :---: | :---: |
| **@Controller** | 组合注解（组合了@Component注解），应用在MVC层（控制层）,DispatcherServlet会自动扫描注解了此注解的类，然后将web请求映射到注解了@RequestMapping的方法上。 |
| **@Service** | 组合注解（组合了@Component注解），应用在service层（业务逻辑层） |
| **@Repository** | 组合注解（组合了@Component注解），应用在dao层（数据访问层） |
| **@Component** | 表示一个带注释的类是一个“组件”，成为Spring管理的Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component还是一个元注解。 |
| **@Autowired** | 表示一个带注释的类是一个“组件”，成为Spring管理的Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component还是一个元注解。 |
| **@Configuration** | 声明当前类是一个配置类（相当于一个Spring配置的xml文件） |
| **@ComponentScan** | 自动扫描指定包下所有使用@Service,@Component,@Controller,@Repository的类并注册 |
| **@Bean** | 注解在方法上，声明当前方法的返回值为一个Bean。返回的Bean对应的类中可以定义init()方法和destroy()方法，然后在@Bean(initMethod=“init”,destroyMethod=“destroy”)定义，在构造之后执行init，在销毁之前执行destroy。 |
| **@Aspect** | 声明一个切面（就是说这是一个额外功能） |
| **@After** | 后置建言（advice），在原方法前执行。 |
| **@Before** | 前置建言（advice），在原方法后执行。 |
| **@Around** | 环绕建言（advice），在原方法执行前执行，在原方法执行后再执行（@Around可以实现其他两种advice） |
| **@PointCut** | 声明切点，即定义拦截规则，确定有哪些方法会被切入 |
| **@Transactional** | 声明事务（一般默认配置即可满足要求，当然也可以自定义） |
| **@Cacheable** | 声明数据缓存 |
| **@EnableAspectJAutoProxy** | 开启Spring对AspectJ的支持 |
| **@Value** | 值的注入。经常与Sping EL表达式语言一起使用，注入普通字符，系统属性，表达式运算结果，其他Bean的属性，文件内容，网址请求内容，配置文件属性值等等 |
| **@PropertySource** | 指定文件地址。提供了一种方便的、声明性的机制，用于向Spring的环境添加PropertySource。与@configuration类一起使用。 |
| **@EnableScheduling** | 注解在配置类上，开启对计划任务的支持。 |
| **@Scheduled** | 注解在方法上，声明该方法是计划任务。支持多种类型的计划任务：cron,fixDelay,fixRate |
| **@Enable*** | 通过简单的@Enable_来开启一项功能的支持。所有@Enable_注解都有一个@Import注解，@Import是用来导入配置类的，这也就意味着这些自动开启的实现其实是导入了一些自动配置的Bean(1.直接导入配置类2.依据条件选择配置类3.动态注册配置类) |
| **@RequestMapping** | 用来映射web请求（访问路径和参数），处理类和方法的。可以注解在类和方法上，注解在方法上的@RequestMapping路径会继承注解在类上的路径。同时支持Serlvet的request和response作为参数，也支持对request和response的媒体类型进行配置。其中有value(路径)，produces(定义返回的媒体类型和字符集)，method(指定请求方式)等属性。 |
| **@GetMapping** | GET方式的@RequestMapping |
| **@PostMapping** | POST方式的@RequestMapping |
| **@ResponseBody** | 将返回值放在response体内。返回的是数据而不是页面 |
| **@RequestBody** | 允许request的参数在request body中，而不是在直接链接在地址的后面。此注解放置在参数前。 |
| **@PathVariable** | 放置在参数前，用来接受路径参数。 |
| **@RestController** | 组合注解，组合了@Controller和@ResponseBody,当我们只开发一个和页面交互数据的控制层的时候可以使用此注解。 |
| **@ModelAttribute** | 将键值对添加到全局，所有注解了@RequestMapping的方法可获得次键值对（就是在请求到达之前，往model里addAttribute一对name-value而已）。 |
| **@SpingBootApplication** | SpringBoot的核心注解，主要目的是开启自动配置。它也是一个组合注解，主要组合了@EnableAutoConfiguration（核心）和@ComponentScan。可以通过@SpringBootApplication(exclude={想要关闭的自动配置的类名.class})来关闭特定的自动配置。 |
| **@ConfigurationProperties** | 将properties属性与一个Bean及其属性相关联，从而实现类型安全的配置。 |
| **@EnableConfigurationProperties** | 注解在类上，声明开启属性注入，使用@Autowired注入。例：@EnableConfigurationProperties(HttpEncodingProperties.class)。 |



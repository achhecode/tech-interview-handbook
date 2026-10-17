---
title: Annotations Cheat Sheet
parent: Spring Boot
nav_order: 2
---

# Annotations Cheat Sheet
{: .no_toc }

Most-used annotations grouped by purpose.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## List the most important Spring and Spring Boot annotations.

Easy
{: .label .label-green }

**Answer**

| Area | Annotations |
|:--|:--|
| Core | `@SpringBootApplication`, `@Configuration`, `@Bean`, `@Component`, `@Service`, `@Repository`, `@Autowired`, `@Qualifier`, `@Primary`, `@Lazy` |
| Web | `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestParam`, `@RequestBody`, `@ResponseStatus`, `@RestControllerAdvice`, `@ExceptionHandler` |
| Data | `@Entity`, `@Id`, `@Transactional`, `@Query` |
| Config | `@Value`, `@ConfigurationProperties`, `@Profile`, `@ConditionalOnProperty` |
| Async and cache | `@Async`, `@EnableAsync`, `@Scheduled`, `@Cacheable`, `@CacheEvict` |
| Testing | `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, `@MockitoBean` |

---

<!--
ADD NEW QUESTIONS ABOVE THIS COMMENT. Copy the block below, then replace the text.
Levels: Easy (label-green), Medium (label-yellow), Hard (label-red).

## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use tables, lists and ```java code blocks.

**Key points**
- Point one

**Follow-up questions**
- Follow-up one

Separate questions with a horizontal rule (three dashes on their own line).
-->

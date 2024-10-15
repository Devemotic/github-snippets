### Entretien technique - Backend - 3

Peux-tu citer au moins 1 problème grave dans le snippet ci-dessous ? On se basera sur les hypothèses suivantes :

-   On supposera que le code compile et que l'indentation est correcte
-   Le webservice présent a pour but de fusionner 3 PDF et retourner le PDF généré

Tu peux également citer des choses que tu vois et que tu connais, ou mentionner des choses qui manquent selon toi. Le but est de donner ton avis sur le code (et de décrire la faille principale).

---

**ByteStreams.java**

```java
// package

// imports

public class ByteStreams {

    // [...]

 /**
   * Reads all bytes from an input stream into a byte array. Does not close the stream.
   *
   * @param in the input stream to read from
   * @return a byte array containing all the bytes from the stream
   * @throws IOException if an I/O error occurs
   */
  public static byte[] toByteArray(InputStream in) throws IOException {
    checkNotNull(in);
    return toByteArrayInternal(in, new ArrayDeque<byte[]>(TO_BYTE_ARRAY_DEQUE_SIZE), 0);
  }


    // [...]

}
```

---

**SimulationResource.java**

```java
package fr.emotic.myclient.myproject.simulation.controller;

// plein d'imports :)

/**
 * REST controller for managing {@link fr.emotic.myclient.myproject.simulation.Simulation}.
 */
@Slf4j
@RestController
@RequestMapping("/api")
@Api(value = "Simulation", tags = {"Simulation"})
public class SimulationResource {

    private static final String ENTITY_NAME = "simulation";

    private final SimulationService simulationService;

    private final SecurityService securityService;
    private final ClientService clientService;

    public SimulationResource(SimulationService simulationService, SecurityService securityService, ClientService clientService) {
        this.simulationService = simulationService;
        this.clientService = clientService;
        this.securityService = securityService;
    }

    // [...]

    @GetMapping("/simulations/{id}/pdf-vendeur")
    public ResponseEntity<byte[]> getPdfVendeur(@PathVariable UUID id) throws IOException {
        log.debug("REST request to get Simulation's pdf vendeur : {}", id);

        if (!SecurityService.userCanAccessSimulation(id)) {
            throw new ResourceForbiddenException();
        }

        return ResponseEntity.ok()
            .headers(HeaderUtil.createContentDisposition("Simulation_vendeur.pdf"))
            .body(this.simulationService.downloadSyntheseVendeur(id));
    }

}
```

---

**SimulationService.java**

```java
package fr.emotic.myclient.myproject.simulation.service;

// Plein d'imports :)

/**
 * Service Implementation for managing {@link Simulation}.
 */
@Slf4j
@Service
@Transactional
public class SimulationService extends QueryService<Simulation> {

    private final PDFService pdfService;
    private final ApplicationProperties applicationProperties;

    public SimulationService(PDFService pdfService, ApplicationProperties applicationProperties) {
        this.pdfService = pdfService;
        this.applicationProperties = applicationProperties;
    }

    // [...]

    public byte[] downloadSyntheseVendeur(UUID id) throws IOException {
        ClassPathResource classPathResource = new ClassPathResource("pdf/body.pdf", this.getClass().getClassLoader());
        ClassPathResource classPathFooterResource = new ClassPathResource("pdf/footer.pdf", this.getClass().getClassLoader());

        InputStream body = classPathResource.getInputStream();
        InputStream footer = classPathFooterResource.getInputStream();

        String url = this.applicationProperties.getHtml2pdfFrontendVendeur(); // jamais null
        byte[] doc1 = ByteStreams.toByteArray(body);
        byte[] doc2 = this.pdfService.html2pdf(url + id.toString());
        byte[] merge1 = this.pdfService.merge(doc1, doc2);
        byte[] doc3 = ByteStreams.toByteArray(footer);
        return this.pdfService.merge(merge1, doc3);
    }

}
```

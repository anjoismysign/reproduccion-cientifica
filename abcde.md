### contexto resumido
deepseek hizo de dominio publico un approach que endogamicamente generaba generalizacion a base de mas generalizacion.
haciendo analogia al adn, la endogamia iba eventualmente a podrir los futuros conocimientos, ya que existe la probabilidad de reproducir una mala generacion.
esto se soluciona con una ampliacion al gan, existiendo mas de un discriminador. para temas relacionados a logica (matematica o desarrollo de software), se puede emplear el uso de compiladores y practicas de desarrollo seguro, como unit testing o incluso retroalimentando a generaciones fallidas mediante el uso de stacktraces. para conocimientos no tan practicos, se pueden emplear SMLs especializados en tareas determinadas, o incluso el uso de un RAG o search engine para verificacion de datos, y eventualmente un ultimo discriminador iba a decidir si era correcto, basandose en la finita linea de si el lenguaje empleado por estos modelos auto reflectivos y de aprendizaje continuo, sigue manteniendo el significado original al lenguaje vernaculo actual, el lenguaje del pueblo.
el uso de sistemas agenticos dentro del aprendizaje continuo, permite generar multiples variaciones de generaciones mas complejas. por ejemplo, en lugar de nut
rir "snapshots" futuros a base de mero aprendizaje reforzado junto a encadenados de pensamiento, a base de meros ejemplos i/o, siendo la entrada peticiones de un humano comun y corriente, sin ningun dominio de terminos especificos dentro del campo, podria no solo pedir exactamente que desea, pero incluso la aplicacion de lenguaje vulgar dentro del proceso.
esto es lo que hacia falta para lo que, de humor post ironico, el humano solo se dedique a describir lo que desea ver, y conforme se van realizando bocetos, es capaz de identificar si desea un cambio.

ejemplo genie/lamp usando sistemas agenticos
![lamp](https://github.com/user-attachments/assets/de8d18ed-5073-47a0-9be9-4129b6c0be35)

ejemplo de genie/lamp en un unico agente (uso de tools por ejemplo)
![agiene](https://github.com/user-attachments/assets/232d0e5f-d9e7-4ec0-bd73-5fb5f7d909e3)

### contexto no resumido/generalizado

- https://www.youtube.com/watch?v=3nM5R23eGkE (timestamp: 15:55 - 16:24, explican en espaniol la recursividad de montecarlo tree search + reinforcement learning)
- [el paper](https://github.com/deepseek-ai/DeepSeek-R1/blob/main/DeepSeek_R1.pdf)

**source original**

https://www.dropbox.com/scl/fi/0e5ck71rsfbb6lqfqfd5q/Archive.zip?rlkey=pw0vo1pwcreqxh47w567tm821&st=989vnkca&dl=0

**recuerden cambiar los LanguageModel para que usen sus propias api key, en especial porque todo esto es de dominio público**

### api sample
```java
package dev.hyperfocus.api;

import dev.hyperfocus.api.agent.AgentType;
import dev.hyperfocus.api.agent.analysis.Analysis;
import dev.hyperfocus.api.agent.factory.JavaAgentFactory;
import dev.hyperfocus.api.entities.RotaryLanguageModel;
import org.jetbrains.annotations.NotNull;

public class Main {
    private static final JavaAgentFactory AGENT_FACTORY = JavaAgentFactory.getInstance();

    private static final RotaryLanguageModel ROTARY_LANGUAGE_MODEL = RotaryLanguageModel.getInstance();

    /**
     * IGNOREN CADA REFERENCIA RELACIONADA A STEPS!!!!!!!!!!!
     */

    /**
     * Por lo general, vas a usar RotaryLanguageModel#getStrong,
     * solo usa los "weak" siempre y cuando se les pida un method pequeño,
     * ojalá un method menor a 12 líneas.
     * Se pueden usar weak models para tareas mas complejas siempre y cuando sean especializados
     * en desarrollo de software, por ejemplo Qwen Coder 2.5 0.5-7 billion parameters.
     * <p>
     * En este caso, como me gusta usar la inferencia "instantanea", me tengo que conformar con los de Llama 3
     */

    public static void main(String[] a) {

        String prompt = """
                Cuando un jugador doma un caballo, se le equipa automaticamente una silla de montar al caballo,
                luego, mientras un caballo sea domado, solo el jugador que es dueño puede montarlo, los demas se bajan 10 ticks despues.
                """;

        /**
         * los steps (el que suele ser el ultimo parametro) son la cantidad de veces que se le permite al agente realizar un intento.
         * creo que la primera vez que ejecutaba, sumaba uno, por lo tanto, poner '3' como steps, le otorga dos intentos de correción,
         * al tercero se detiene y le devuelve al usuario/programador el desarrollo del sistema agentico, por lo que mediante
         * prompt "engineering" se podría corregir el problema en cualquier caso futuro,
         * ya que solo se debería optimizar el sistema agentico si es necesaria una nueva
         * solucion determinista
         */
        AgentType outliner = AGENT_FACTORY.createBukkitProjectOutliner(prompt, ROTARY_LANGUAGE_MODEL.getStrong(), 3); //solo lo deja corregirse un maximo de 2 veces
        Analysis outlinerAnalysis;
        try {
            outlinerAnalysis = outliner.createAnalysisProgress().getAnalysis();
        } catch ( Throwable throwable ) {
            throwable.printStackTrace();
            return;
        }
//        if (!outlinerAnalysis.getStatus().isFine()) {
//            outlinerAnalysis.debug();
//            return;
//        }
        //de aca para abajo, deberia de generar de manera correcta el outline para el resto de sistemas agenticos
        //sin embargo, acabo de recordar que los "steps" solo son usados en agentes con compilador
        //que es equivalente a que si o si, el Analysis#getStatus deberia de ser siempre FINE
        String outline = outlinerAnalysis.getFeedback();

        AgentType bukkitDev = AGENT_FACTORY.createBukkitPluginDeveloper(outline, ROTARY_LANGUAGE_MODEL.getStrong(), 3);
        Analysis bukkitDevAnalysis = bukkitDev.createAnalysisProgress().getAnalysis();
        String sourceCode = bukkitDevAnalysis.getFeedback();

        int compilerMaxSteps = 3;

        //le pongo 1 step porque quiero que si el compilador falle, no intente corregir más de una vez
        AgentType compilerTest = AGENT_FACTORY.createTester(sourceCode, ROTARY_LANGUAGE_MODEL.getWeak(), compilerMaxSteps);
        Analysis compilerTestAnalysis;
        try {
            compilerTestAnalysis = compilerTest.createAnalysisProgress().getAnalysis();
        } catch ( Throwable throwable ) {
            throwable.printStackTrace();
            System.out.println("no compiler test analysis: " + throwable.getMessage());
            System.out.println(sourceCode);
            return;
        }
        if (!compilerTestAnalysis.getStatus().isFine()) {
            compilerMaxSteps--;
            //ACÁ DEBERÍA DE DECIR EL STACKTRACE
            String stacktrace = compilerTestAnalysis.getFeedback();
            fix(stacktrace, sourceCode, sourceCode, compilerMaxSteps);
        }
        System.out.println("seems fine, assure QA");
        System.out.println(sourceCode);
    }

    private static void fix(@NotNull String error,
                            @NotNull String liveSourceCode,
                            @NotNull String originalSourceCode,
                            int ladderSteps) {
        Analysis bukkitPluginFixerAnalysis;
        try {
            bukkitPluginFixerAnalysis = AGENT_FACTORY.createBukkitPluginFixer(error, liveSourceCode, originalSourceCode, ROTARY_LANGUAGE_MODEL.getStrong(), ladderSteps)
                    .createAnalysisProgress().getAnalysis();
        } catch ( Throwable throwable ) {
            throwable.printStackTrace();
            System.out.println("no bukkitPluginFixer analysis: " + throwable.getMessage());
            System.out.println(liveSourceCode);
            return;
        }
        liveSourceCode = bukkitPluginFixerAnalysis.getFeedback();
        if (bukkitPluginFixerAnalysis.getStatus().isFine()) {
            System.out.println("seems fine, assure QA");
            //EL CÓDIGO FUENTE QUE PUEDE COMPILAR
            System.out.println(liveSourceCode);
            return;
        }

        Analysis compilerTestAnalysis;
        try {
            compilerTestAnalysis = AGENT_FACTORY.createTester(liveSourceCode, ROTARY_LANGUAGE_MODEL.getWeak(), ladderSteps).createAnalysisProgress().getAnalysis();
        } catch ( Throwable throwable ) {
            throwable.printStackTrace();
            System.out.println("no compiler test analysis: " + throwable.getMessage());
            System.out.println(liveSourceCode);
            return;
        }
        if (!compilerTestAnalysis.getStatus().isFine()) {
            ladderSteps--;
            if (ladderSteps <= 0) {
                //ACÁ SE DETIENE EL SISTEMA AGÉNTICO
                compilerTestAnalysis.debug();
                return;
            }

            //ACÁ DEBERÍA DE DECIR EL STACKTRACE
            String stacktrace = compilerTestAnalysis.getFeedback();
            //ACÁ DEBERÍA DE PASAR AL LOGGER CADA DATO POSIBLE, COMO CONTEXTO PARA UN HUMANO/DESARROLLADOR
            compilerTestAnalysis.debug();
            fix(stacktrace, liveSourceCode, originalSourceCode, ladderSteps);
            return;
        }

        String compilable = compilerTestAnalysis.getFeedback();
        System.out.println("seems fine, assure QA");
        System.out.println(compilable);

    }
}
```

### solo cambien en el builder, el que diga apiKey
```java
package dev.hyperfocus.api.entities;

import dev.langchain4j.model.Tokenizer;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiChatModel;
import dev.langchain4j.model.openai.OpenAiTokenizer;

import java.time.Duration;

public enum LanguageModel {
    CEREBRAS_LLAMA3_1_8B(OpenAiChatModel.builder()
            .baseUrl("https://api.cerebras.ai/v1")
            .apiKey("csk-k8m5vd34pn8jp93hrxnfpv44f6f8pevxh8jh3kknxxkc8krf")
            .modelName("llama3.1-8b")
            .maxTokens(8192)
            .timeout(Duration.ofSeconds(60))
            .build(), 128000),
    CEREBRAS_LLAMA3_1_70B(OpenAiChatModel.builder()
            .baseUrl("https://api.cerebras.ai/v1")
            .apiKey("csk-k8m5vd34pn8jp93hrxnfpv44f6f8pevxh8jh3kknxxkc8krf")
            .modelName("llama3.1-70b")
            .maxTokens(8192)
            .timeout(Duration.ofSeconds(60))
            .build(), 128000),
    SAMBANOVA_LLAMA3_1_8B(OpenAiChatModel.builder()
            .baseUrl("https://api.sambanova.ai/v1")
            .apiKey("8ac63b2b-c545-48c2-862d-eac714eb908c")
            .modelName("Meta-Llama-3.1-8B-Instruct")
            .maxTokens(8192)
            .timeout(Duration.ofSeconds(60))
            .build(), 128000),
    SAMBANOVA_LLAMA3_1_70B(OpenAiChatModel.builder()
            .baseUrl("https://api.sambanova.ai/v1")
            .apiKey("8ac63b2b-c545-48c2-862d-eac714eb908c")
            .modelName("Meta-Llama-3.1-70B-Instruct")
            .maxTokens(8192)
            .timeout(Duration.ofSeconds(60))
            .build(), 128000);

    private static final Tokenizer openAI = new OpenAiTokenizer();
    private final ChatLanguageModel model;
    private final int tokens;

    LanguageModel(
            ChatLanguageModel model,
            int tokens) {
        this.model = model;
        this.tokens = tokens;
    }

    public static Tokenizer getOpenAITokenizer() {
        return openAI;
    }

    public ChatLanguageModel getChatLanguageModel() {
        return model;
    }

    public int getTokens() {
        return tokens;
    }
}
```



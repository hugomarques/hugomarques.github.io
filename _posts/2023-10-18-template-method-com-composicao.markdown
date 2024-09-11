---
title: Template Method com composição
published: true
description: Como atingir o mesmo objetivo do Template Method através de composição em vez de herança. 
tags: #designpatterns #composicao #heranca #java
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2023-10-18 22:13 +0000
---

Eu acho que esse vai ser curto, vamos lá! Hoje mais cedo eu fiz o seguinte comentário sobre herança no finado Twitter.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1x1a8e8081ofzx2fx1p3.png)

Imediatamente, surgiram alguns comentários interessantes, mas um dele me chamou a atenção. O amigo @DevCorinthiano777 perguntou:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/41qi62r3k6u5jrpk8o7m.png)

E com isso, automaticamente as engrenagens internas aqui na minha cabeça começaram a funcionar 🤔.

## Mas o que danado é um Template Method?

Sendo super sucinto, o Template Method é um "design pattern" (ou padrão de projeto em PT-br), que define uma "receita de bolo". Essa receita é definida na super-classe e a partir daí as classes filhas podem implementar passos específicos dessa "receita". Isso fica mais claro no exemplo abaixo:

Disclaimer: Eu sou um dev cansado, usei o gepeto pra me ajudar a escrever o exemplo rápido.

```Java
abstract class ReviewTemplate {
    private String reviewText;

    public ReviewTemplate(String reviewText) {
        this.reviewText = reviewText;
    }

    // Método de template que orquestra o processo de validação e salvamento
    public void processReview() {
        if (validateReview()) {
            saveReview();
            System.out.println("Revisão salva com sucesso.");
        } else {
            System.out.println("A revisão não é válida e não foi salva.");
        }
    }

    // Método abstrato que deve ser implementado pelas subclasses para a validação
    protected abstract boolean validateReview();

    // Método comum para salvar a revisão (implementação padrão)
    private void saveReview() {
        System.out.println("Revisão salva no banco de dados: " + reviewText);
    }
}

class CriarReview extends ReviewTemplate {
    public CriarReview(String reviewText) {
        super(reviewText);
    }

    @Override
    protected boolean validateReview() {
        // Implementação específica de validação para criar uma revisão
        return reviewText != null && !reviewText.isEmpty();
    }
}

class AtualizarReview extends ReviewTemplate {
    public AtualizarReview(String reviewText) {
        super(reviewText);
    }

    @Override
    protected boolean validateReview() {
        // Implementação específica de validação para atualizar uma revisão
        return reviewText != null && !reviewText.isEmpty();
    }
}

public class Main {
    public static void main(String[] args) {
        // Criar uma nova revisão
        ReviewTemplate criarReview = new CriarReview("Esta é uma ótima revisão.");
        criarReview.processReview();

        // Atualizar uma revisão existente
        ReviewTemplate atualizarReview = new AtualizarReview("Versão atualizada da revisão.");
        atualizarReview.processReview();
    }
}
```

Observe como o review template define um padrão de como todas as suas filhas são tratadas. A partir daí as classes filhas só seguem esse padrão, implementando pedaços do algoritmo e expondo apenas o método público `processReview()`

## Entendi, mas se a implementação usa herança, como tu vai fazer pra usar composição?

Simples, aí que entra a mágica da injeção de dependências. Olha que bacana:

```Java
interface ReviewValidator {
   
    public boolean validateReview();

}

class CriarReviewValidator implements ReviewValidator {
    private String reviewText;

    public ReviewValidator(String reviewText) {
        this.reviewText = reviewText;
    }

    public boolean validateReview() {
        // Lógica de validação comum para todas as revisões
        return reviewText != null && !reviewText.isEmpty();
    }
}

class AtualizarReviewValidator implements ReviewValidator {
    private String reviewText;

    public ReviewValidator(String reviewText) {
        this.reviewText = reviewText;
    }

    public boolean validateReview() {
        // Lógica de validação comum para todas as revisões
        return reviewText != null && !reviewText.isEmpty();
    }
}

class ReviewWriter {
    private ReviewValidator validator;

    public ReviewWriter(ReviewValidator validator) {
        this.validator = validator;
    }

    public void writeReview() {
        if (validator.validateReview()) {
            saveReview();
            System.out.println("Revisão salva com sucesso.");
        } else {
            System.out.println("A revisão não é válida e não foi salva.");
        }
    }

    private void saveReview() {
        System.out.println("Revisão salva no banco de dados: " + validator.reviewText);
    }
}

public class Main {
    public static void main(String[] args) {
        // Criar uma nova revisão
        ReviewWriter criarReview = new ReviewWriter(new CriarReviewValidator("Essa não é uma boa review"));
        criarReview.writeReview();

        // Atualizar uma revisão existente
        ReviewWriter atualizarReview = new ReviewWriter(new AtualizarReviewValidator("Versão atualizada da revisão."));
        atualizarReview.writeReview();
    }
}
```

Viu como dá pra fazer o mesmo comportamento usando apenas composição de objetos?

Tá blz, entendi. Mas e qual a vantagem disso? 

Com herança, você tem um alto acoplamento entre a super-classe e suas filhas. Se você faz uma mudança na super-classe, todas as classes filhas são afetadas quer você queira ou não.

Com composição, os comportamentos ficam todos mais isolados. É possível criar novos comportamentos e inseri-los ou removê-los conforme você desejar e você não corre o risco de afetar os `validators` caso você altere o `ReviewWriter`.

## Conclusão

Esse post foi escrito de forma super rápida, só porque eu acho o assunto interessante e fazia muito tempo que eu tinha postado algo com código. Eu espero que vocês tenham curtido o assunto. 

Se foi o caso, deixa aí o like e comentário que isso me incentiva a escrever mais sobre esse tipo de assunto com código.

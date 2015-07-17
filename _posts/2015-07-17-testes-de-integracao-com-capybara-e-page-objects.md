---
layout: post
title: Testes de Integração com Capybara e Page Objects
post_author: Jaison Erick
post_gravatar: 9d7cd56baf4cd888f1fcb551132691ad
---

Clareza no código não deve ser uma virtude apenas do código de produção, mas
também dos testes. Testes mal escritos ou difíceis de ler acabam sendo deixados
de lado e, frequentemente, corrigidos apenas para passar uma nova implementação.
Além dos problemas mais óbvios, como não testar o que deve ser testado, eles
podem acabar gerando falsos positivos, destruindo a confiança que a equipe tem
em sua suíte de testes.

Embora não seja incomum que testes complexos sejam escritos em nível unitário,
eles são muito mais frequentes em testes de aceitação ou de integração,
justamente por envolverem uma porção maior da aplicação. Talvez Por este motivo
a maior parte das ferramentas que facilitam a leitura e escrita dos testes
tenham foco neste tipo de teste.

<!-- more -->

O seguinte exemplo mostra um caso de teste de aceitação, utilizando o Capybara:

{% highlight ruby %}
feature 'create a new ad segment' do
  given(:admin) { create :admin }

  scenario 'with teenagers', js: true do
    create_list :user, 5, age: 15
    create_list :user, 2, age: 25
    create_list :user, 2, age: 13
    create_list :user, 2, age: 19
    create_list :user, 4, age: 12

    @total_teens = 9
    @total_users = 15

    sign_in_as :admin

    visit new_ad_path
    expect(page).to have_content('Create AD segment')

    find('.segment .age').trigger('click')
    expect('.segment-body').to have_contents('Setup age segment')

    page.execute_script("$('#age-slider').slider('values', [13,19]")

    expect('.segment-stats').to have_content('Ages: 13-19')

    expect('.segment-total .selected').to have_content(@total_teens)

    fill_in('AD set name', with: 'Teenagers')

    click_button('Save')

    expect(find('.status-_bar')).to have_content("Ad set 'Teenagers' created")
  end
end
{% endhighlight %}

É possível ver que o código de teste é bastante extenso, o que acontece de ser
comum em testes de aceitação, porém por qual motivo criticamos um código extenso
para produção e não temos o mesmo nível de cuidado com um teste? Afinal o
propósito da crítica deveria ser o mesmo: Facilitar a leitura e a manutenção.
Este exemplo já utiliza alguns recursos que facilitam a leitura, como o Factory
Girl e a utilização de helpers. Perceba que o setup inicial dos dados é bastante
complexo, pois como é um teste de integração, precisa envolver diferentes fontes
de dados. O helper "sign_in_as" já é resultado de um refactoring neste teste,
simplificando o processo de login na aplicação.

Um primeiro ponto para facilitar a leitura, é retirar as chamadas de
configuração dos dados do cenário. Embora este setup seja focado para este
cenário específico, ter 8 linhas de código apenas para o setup do banco é um
desperdício e dificulta a leitura. Uma forma é extrair o setup para um método:

{% highlight ruby %}
feature 'create a new ad segment' do
  given(:admin) { create :admin }

  scenario 'with teenagers', js: true do
    setup_teenager_set
    # ...
  end

  def setup_teenager_set
    create_list :user, 5, age: 15
    create_list :user, 2, age: 25
    create_list :user, 2, age: 13
    create_list :user, 2, age: 19
    create_list :user, 4, age: 12

    @total_teens = 9
    @total_users = 15
  end
end
{% endhighlight %}

Com o setup realizado, podemos começar a trabalhar na extração de métodos que
facilitem o controle da interface. Neste cenário, acessamos
a tela de criação de novos segmentos para anúncios, configuramos o intervalo de
idades que queremos para este segmento, confirmamos que o resultado do set é o
esperado e finalmente salvamos o conjunto. Nosso objetivo é tornar a leitura
deste teste tão simples quanto a descrição do comportamento que esperamos da
aplicação.

Uma forma para alcançar este objetivo é escrever, na forma de comentários, este
comportamento:

{% highlight ruby %}
feature 'create a new ad segment' do
  given(:admin) { create :admin }

  scenario 'with teenagers', js: true do
    setup_teenager_set
    sign_in_as :admin

    # Open create segment page
    visit new_ad_path
    expect(page).to have_content('Create AD segment')

    # Open the Age Segment Configurator
    find('.segment .age').trigger('click')
    expect('.segment-body').to have_contents('Setup age segment')

    # Configure Age Range
    page.execute_script("$('#age-slider').slider('values', [13,19]")
    expect('.segment-stats').to have_content('Ages: 13-19')

    # Expect selected total
    expect('.segment-total .selected').to have_content(@total_teens)

    # Fill ad set name
    fill_in('AD set name', with: 'Teenagers')

    # Save
    click_button('Save')

    # Expect status message to be "Ad set created"
    expect(find('.status-bar')).to have_content("Ad set 'Teenagers' created")
  end

  # ...
end
{% endhighlight %}

Neste momento podemos criar uma nova classe para controlar a nossa tela de
criação de segmentos. Vamos criar uma classe chamada "SegmentPage" na pasta
"spec/pages/segment_page.rb"

{% highlight ruby %}
class SegmentPage
end
{% endhighlight %}

A primeira substituição que podemos fazer é extrair a ação de abrir o "Age
Segment Configurator" para o nosso page object:

{% highlight ruby %}
feature 'create a new ad segment' do
  given(:page_object) { SegmentPage.new }
  # ...
  scenario 'with teenagers', js: true do
    # ...
    # Open the Age Segment Configurator
    segment_page.open_age_segment_configurator
    # ...
  end
  # ...
end
{% endhighlight %}

E em nosso Page Object:

{% highlight ruby %}
class SegmentPage
  def open_age_segment_configurator
    find('.segment .age').trigger('click')
    expect('.segment-body').to have_contents('Setup age segment')
  end
end
{% endhighlight %}

Com esta extração, quem ler nosso código imediatamene saberá o que está
aconecendo "Abra o Age Segment Configurator", e não, "clique no botão com a
classe XYZ".

Seguindo neste processo, podemos extrair todas as ações com o seguinte
resultado:

{% highlight ruby %}
feature 'create a new ad segment' do
  # ...
  scenario 'with teenagers', js: true do
    sign_in_as :admin

    segment_page.go_to_create
    segment_page.open_age_segment_configurator
    segment_page.set_age_range(from: 13, to: 19)

    # Expect selected total
    expect('.segment-total .selected').to have_content(@total_teens)

    segment_page.fill_name('Teenagers')
    segment_page.save

    # Expect status message to be "Ad set created"
    expect(find('.status-bar')).to have_content("Ad set 'Teenagers' created")
  end
end
{% endhighlight %}

O cenário é muito mais simples de ler, inclusive sendo possível remover os
comentários. Outra vantagem é que todos os seletores agora estão contidos na
classe SegmentPage e em uma eventual melhoria visual que venha a quebrar este
teste, apenas um lugar precisa ser atualizado.

Substituir as ações já foi um grande ganho, porém podemos ir uma etapa adiante,
extraindo o restante dos seletores CSS. O resultado final ficou o seguinte:

{% highlight ruby %}
feature 'create a new ad segment' do
  given(:admin) { create :admin }

  scenario 'with teenagers', js: true do
    setup_teenager_set
    sign_in_as :admin

    segment_page.go_to_create
    segment_page.open_age_segment_configurator
    segment_page.set_age_range(from: 13, to: 19)

    expect(segment_page.total_selected).to have_contents(@total_teens)

    segment_page.fill_name('Teenagers')
    segment_page.save

    expect(segment_page.status_bar).to have_content("Ad set 'Teenagers' created")
  end

  def setup_teenager_set
    create_list :user, 5, age: 15
    create_list :user, 2, age: 25
    create_list :user, 2, age: 13
    create_list :user, 2, age: 19
    create_list :user, 4, age: 12

    @total_teens = 9
    @total_users = 15
  end
end
{% endhighlight %}


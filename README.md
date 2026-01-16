# Planogram-magaza-urun-yonetimi
![](docs/src/figures/planogram.svg)

[![Docs Image](https://img.shields.io/badge/docs-latest-blue.svg)](https://gamma-opt.github.io/ShelfSpaceAllocation.jl/dev/)
![Runtests](https://github.com/gamma-opt/ShelfSpaceAllocation.jl/workflows/Runtests/badge.svg)

Bu paket, perakende mağazaları bağlamında *raf alanı dağıtım problemini (SSAP)* çözmek için geliştirilmiş bir optimizasyon modeli içerir ve *karma tamsayı doğrusal programlama (MILP)* olarak formüle edilmiştir. Paketi hem modeli geliştirmek hem de çalıştırmak için tasarladık. Model, görselleştirme yetenekleri, girdi/çıktı ile ilgili fonksiyonlar ve örnek senaryoları içerir. Dokümantasyon, paketin nasıl kullanılacağını, işlevlerini ve modeli ayrıntılı olarak kapsamaktadır.

Bu paket, Aalto Üniversitesi Sistem Analizi Laboratuvarı'ndaki bir araştırma projesinin parçasıdır ve *Fabricio Oliveira* ile *Jaan Tollander de Balsch* tarafından hazırlanmıştır.


## Örnekler
|Özellik|Küçük|Orta|Büyük|
|---------|-----|-------|------|
|Ürünler  |118  |221    |193   |
|Raflar   |7    |7      |10    |
|Bloklar  |7    |9      |23    |
|Modüller |1    |1      |2     |

Yukarıdaki tabloda boyutları açıklanan `küçük`, `orta` ve `büyük` olmak üzere üç örnek durum mevcuttur. Aşağıdaki örnek betik, `PlanogramMagazaUrunYonetimi.jl` ve Gurobi optimizasyonu kullanılarak bu örneklerin nasıl çözüleceğini göstermektedir.

```julia
using Dates, JuMP, Gurobi
using PlanogramMagazaUrunYonetimi

case = "small"
output_dir = "examples/output/$case/$(string(Dates.now()))"
mkpath(output_dir)

parameters = Params(
    "examples/instances/$case/products.csv",
    "examples/instances/$case/shelves.csv"
)
specs = Specs(height_placement=true, blocking=true)
model = ShelfSpaceAllocationModel(parameters, specs)

optimizer = optimizer_with_attributes(
    () -> Gurobi.Optimizer(Gurobi.Env()),
    "TimeLimit" => 5*60,
    "MIPFocus" => 3,
    "MIPGap" => 0.01,
)
set_optimizer(model, optimizer)
optimize!(model)

variables = Variables(model)
objectives = Objectives(model)
```

Değerlerin JSON olarak kaydedilmesi.
```julia
save_json(specs, joinpath(output_dir, "specs.json"))
save_json(parameters, joinpath(output_dir, "parameters.json"))
save_json(variables, joinpath(output_dir, "variables.json"))
save_json(objectives, joinpath(output_dir, "objectives.json"))
```

Değerlerin JSON'dan yüklenmesi.
```julia
specs = load_json(Specs, joinpath(output_dir, "specs.json"))
parameters = load_json(Params, joinpath(output_dir, "parameters.json"))
variables = load_json(Variables, joinpath(output_dir, "variables.json"))
objectives = load_json(Objectives, joinpath(output_dir, "objectives.json"))
```

Dokümantasyonun [çizim](https://gamma-opt.github.io/ShelfSpaceAllocation.jl/plotting/) bölümü, sonuçların nasıl görselleştirileceğini göstermektedir.

*Gevşet-ve-sabitle* ve *sabitle-ve-optimize et* sezgisel yöntemlerinin örneği [`heuristics.jl`](./examples/heuristics.jl) dosyasında mevcuttur.

## Kurulum
[Julia dilini](https://julialang.org/) kurun ve ardından bu paketi kurun.

```bash
pkg> add https://github.com/gamma-opt/PLANOGRAM.jl
```

## Geliştirme
Depoyu klonlayın
```bash
git clone https://github.com/gamma-opt/PLANOGRAM.jl
```

Bağımlılıkları kurun
```
pkg> dev .
```

Gurobi gibi bir çözücü kurun.


## Çözücü Kurulumu
JuMP modelini çözmek için uygun bir çözücü seçmek kullanıcıya kalmıştır. Küçük örnekler için GLPK yeterli olabilir, ancak büyük örnekler için Gurobi veya CPLEX gibi ticari bir çözücü önerilir.

Gurobi, ücretsiz akademik lisans sağlayan güçlü bir ticari optimizasyon aracıdır. Gurobi, [`Gurobi.jl`](https://github.com/JuliaOpt/Gurobi.jl) kullanılarak Julia ile arayüzlenebilir. Programı çalıştırmak için Julia ve Gurobi'yi kurma adımları şunlardır:

1) *Gurobi* lisansı alın ve [Gurobi'nin web sitesindeki](http://www.gurobi.com/) talimatları izleyerek Gurobi çözücüsünü kurun.

2) `GUROBI_HOME` ortam değişkeninin Gurobi dizin yoluna ayarlandığından emin olun. Bu, standart kurulumun bir parçasıdır. Gurobi kütüphanesi Unix platformlarında `GUROBI_HOME/lib` altında, Windows'ta ise `GUROBI_HOME\bin` altında aranacaktır. Kütüphane bulunamazsa, sürümünüzün `deps/build.jl` dosyasında listelendiğini kontrol edin. Ortam değişkeni, `.bashrc` dosyasına `export GUROBI_HOME="<yol>/gurobi811/linux64"` eklenerek ayarlanabilir. `<yol>`, platform `linux64` ve sürüm numarası `811`'i Gurobi kurulumunuzdaki değerlerle değiştirin.

3) Aşağıdaki komutları çalıştırarak Julia paket yöneticisine `Gurobi.jl`'yi kurun:
   ```
   pkg> add Gurobi
   pkg> build Gurobi
   ```


## Dokümantasyon
Proje dokümantasyonu [Documenter.jl](https://juliadocs.github.io/Documenter.jl/stable/) kullanılarak oluşturulmuştur. Dokümantasyonu derlemek için `docs` dizinine gidin ve şu komutu çalıştırın:
```bash
julia make.jl
```

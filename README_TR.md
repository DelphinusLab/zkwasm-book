# ZKWASM-Kitap

Bu repo ZKWASM hakkında belgeler içerir ve şu adresten canlı olarak görülebilir: [Kitabı canlı olarak görün](https://zkwasmdoc.gitbook.io/delphinus-zkwasm/).

DelphiusLab'ın misyonu, Web2 geliştiricilerine uygulamalarında Web3'ün gücünden yararlanmaları için özlü bir araç seti sağlamaktır. ZKWASM (Web Assembly'yi destekleyen ZKSNARK sanal makinesi), WASM çalışma zamanında çalışan zengin uygulamalar ile zincir üzerindeki akıllı sözleşmeler arasında güvenilir bir katman görevi görür.

WASM (veya WebAssembly) assembly'ye benzer açık standart bir ikili kod formatıdır. İlk amacı, mevcut web ekosistemi için geliştirilmiş performans ile java-script'e bir alternatif sağlamaktı. Platform bağımsızlığı, ön uç esnekliği (C, C++, assembly script, rust vb. dahil olmak üzere çoğu dilden derlenebilir), iyi izole edilmiş çalışma zamanı ve yerel bir ikilinin hızına yaklaşan hızından yararlanarak, dağıtılmış bulut ve uç bilişimde kullanımı artmaktadır. Son zamanlarda, kullanıcıların AWS Lambda, Open Yurt, AZURE vb. üzerinde özelleştirilmiş işlevler çalıştırması için popüler bir ikili biçim haline gelmiştir.

ZKWASM fikri, SNARG (Succinct non-interactive arguments) ve sıfır bilgi ispatının bir kombinasyonu olan ZKSNARK'tan (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge) türetilmiştir. Genel olarak, ZKSNARK'ın benimsenmesi genellikle aritmetik devrelerde veya devre dostu dillerde (Pinocchio, TinyRAM, Buffet/Pequin, Geppetto, xJsnark framework, ZoKrates) bir programın uygulanmasını gerektirir ve bu da mevcut programların gücünden faydalanması için bir engel oluşturur. Alternatif bir yaklaşım, ZKSNARK'ı kaynak koduna uygulamak yerine, bir sanal makinenin bayt kodu düzeyinde uygulamak ve zksnark destekli bir sanal makine uygulamaktır. Bu çalışmada, tüm WASM sanal makinesini ZKSNARK devrelerinde yazma yaklaşımını benimsiyoruz, böylece mevcut WASM uygulamaları herhangi bir değişiklik yapmadan sadece ZKWASM üzerinde çalışarak ZKSNARK'tan faydalanabilir. Bu nedenle, bulut hizmeti sağlayıcısı herhangi bir kullanıcıya hesaplama sonucunun dürüst bir şekilde hesaplandığını ve hiçbir özel bilginin sızdırılmadığını kanıtlayabilir.

Dokümanlar aşağıdaki kitleler için eğitimler içerir

1. ZKWASM sanal makinesinde çalışan WASM tabanlı uygulamalar geliştirmeye ve zkWASM ana bilgisayar devrelerinin özelliklerine aşina olmak için yeni başlayanlar.
2. Kendi uygulama toplamaları için ZKWASM yeteneğinden yararlanmak isteyen sofistike toplama uygulama üreticileri.
3. WASM'ı daha özelleştirilebilir senaryolar için farklı ZKP devrelerini bağlamak için bir tutkal dili olarak kullanmak isteyen devre entegratörleri (örn. ML, ORACL, VRF, vb.)
4. Kendi ZK sanal makinelerini oluşturmak için WASM bayt kodunun çekirdek minimal setini kullanmak isteyen ZKVM oluşturucuları.

Bu belge aynı zamanda ZKP devre ataması, kanıt gruplama ve birleştirme, devam eden yürütmenin kanıt yapısı, WASM ana devre entegrasyonu ve WASI desteği hakkında bilgi sağlar.


**Bu kitap bir WIP'dir ve sürekli güncellenmektedir.**

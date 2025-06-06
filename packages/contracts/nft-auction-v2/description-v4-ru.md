
# Контракт аукциона за ton и jetton

Этот контракт рекомендуется использовать когда есть много покупателей на одну нфт. Продавец выставляет нфт на аукцион,
а покупатели предлагают свою цену, побеждает покупатель с самой высокой ценой. Каждая последующая должна быть больше предыдущей на
указанный процент (min_step) но не меньше чем на 0.1 тон. Если цена указана в жетонах, то ограничений на следующую ставку нет.
Продавец может ограничить минимальную цену, также продавец может указать максимальную цену, за которую можно сразу выкупить 
нфт с аукциона. Аукцион ограничен по времени рекомендуемая длительность - 3 дня, максимум 19 дней. 
Продавец может отменить аукцион если в нем не было ставок, если есть хотя бы одна ставка то
аукцион будет идти до конца, либо до достижения максимальной ставки. Аукцион имеет защиту от ставки в последний момент,
если ставка делается за 5 минут (этот параметр настраивается, но не более 1 дня) до окончания аукциона, то время аукциона продлевается еще на 5 минут.
Все сообщения на этот контракт должны быть отправлены с баунс флагом, контракт выбрасывает ошибки.

### get_auction_data_v4

1. int activated? -1 аукцион получил нфт и готов к работе 
2. int end? -1 - аукцион завершен 0 - не завершен
3. int end_time timestamp времени окончания аукциона или время фактического завершения аукциона если нфт была продана
4. slice(MsgAddress) mp_addr адрес контракта маркетплейса, этот адрес может отменить аукцион без ставок или завершить его после end_time
5. slice(MsgAddress) nft_addr адрес нфт
6. slice(MsgAddress) nft_owner адрес владельца нфт, этот получит тоны с продажи нфт, может отменить аукцион без ставок или завершить его после end_time
7. int(coins) last_bid сумма последней ставки или 0 если ставки не было
8. slice(MsgAddress) last_member адрес кошелька с которого была сделана последня ставка
9. int min_step процент шага ставки 1 - 100
10. int mp_fee_addr адрес кошелька для комиссии маркетплейса
11. int mp_fee_factor процент комиссии маркетплейса выражен двумя цифрами, пример: mp_fee_factor = 10 mp_fee_base = 100, означает что комиссия маркетплейса 10% 10/100=0.1=10% 
12. int mp_fee_base
13. slice(MsgAddress) royalty_fee_addr адрес кошелька для роялти коллекции
14. int royalty_fee_factor процент роялти коллекции, смотри mp_fee_factor
15. int royalty_fee_base
16. int(coins) max_bid сумма максимальной ставки или 0, если ее нет
17. int(coins) min_bid сумма минимальной ставки
18. int created_at? timestamp времени создания аукциона, используется для генерации разных адресов контрактов
19. int last_bid_at timestamp времени последней ставки или 0 если ставок не было
20. int is_canceled -1 -- означает что аукцион был отменен
21. int step_time кол-во секунд на сколько продлевается аукцион если ставка сделана в последний момент
22. int last_query_id последний обработанный query_id, если он был в запросе
23. slice(MsgAddress) jetton_wallet адрес жетон кошелька, может быть пустой если продажа за тоны
24. slice(MsgAddress) jetton_master адрес мастер контракта жетон кошелька
25. int is_broken_state -1 означает что контракт находится в состоянии когда не может принимать ставки
26. int public_key публичный ключ или 0, этот ключ используется для деплоя смарт контракта с оплатой за жетоны.

### Рекомендуемые проверки перед использованием контракта

- хеш кода контракта должен совпадать с эталонным, нельзя полагаться только на гет методы
- get_auction_data_v4 должен вызываться нормально, без ошибок, activated? == -1 end? = 0 is_broken_state = 0
- процент комиссии маркетплейса и процент роялти коллекции в сумме должны быть меньше 100
- необходимо проверить что нфт, на которую ссылается контракт продажи (nft_address), действительно принадлежит ему, то-есть у нфт owner_address это адрес контракта продажи
- значение min_step в диапазоне от 1 до 100
- значение end_time не больше текущей даты +14 дней
- необходимо проверять валидность jetton_wallet относительно jetton_master, не допускается ситуаций когда есть jetton_master но нет jetton_wallet или наоборот   
- если аукцион за тоны (поля jetton_wallet jetton_master пусты), то баланс контракта в тонах должен быть больше чем last_bid

+++
title = 'test'
date = 2024-08-04T03:25:53+08:00
categories = ["test"]
tags = ["test"]
+++

行内数学公式：$a^2 + b^2 = c^2$。

块公式，

$$
a^2 + b^2 = c^2
$$

<div>
$$
\boldsymbol{x}_{i+1}+\boldsymbol{x}_{i+2}=\boldsymbol{x}_{i+3}
$$
</div>



```Java
//2. 策略查询  
StrategyEntity strategy = repository.queryStrategyEntityByStrategyId(strategyId);  
  
//3. 抽奖前的规则过滤  
RuleActionEntity<RuleActionEntity.RaffleBeforeEntity> ruleActionEntity = this.doCheckRaffleBeforeLogic(RaffleFactorEntity.builder()  
        .userId(userId)  
        .strategyId(strategyId)  
        .build(), strategy.ruleModels());  
if (RuleLogicCheckTypeVO.TAKE_OVER.getCode().equals(ruleActionEntity.getCode())){  
    if (DefaultLogicFactory.LogicModel.RULE_BLACKLIST.getCode().equals(ruleActionEntity.getRuleModel())){  
        //黑名单返回固定奖品id  
        return RaffleAwardEntity.builder()  
                .awardId(ruleActionEntity.getData().getAwardId())  
                .build();  
    } else if (DefaultLogicFactory.LogicModel.RULE_WIGHT.getCode().equals(ruleActionEntity.getRuleModel())) {  
        //根据权重信息返回的进行抽奖  
        RuleActionEntity.RaffleBeforeEntity raffleBeforeEntity = ruleActionEntity.getData();  
        String ruleWeightValueKey = raffleBeforeEntity.getRuleWeightValueKey();  
        Integer awardId = strategyDispatch.getRandomAwardId(strategyId, ruleWeightValueKey);  
        return RaffleAwardEntity.builder()  
                .awardId(awardId)  
                .build();  
    }  
}  
  
//4. 默认抽奖流程  
Integer awardId = strategyDispatch.getRandomAwardId(strategyId);
```
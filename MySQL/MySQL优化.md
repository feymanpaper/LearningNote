### 带排序的分页查询优化
https://www.bilibili.com/video/BV198411F7hx/?spm_id_from=333.337.search-card.all.click&vd_source=108d23f95683578313bdaf5d938b5b3d
1. ﻿﻿﻿浅分页可以给 order by 字段加索引
2. ﻿﻿深分页可以给 order by 和 select 字段加联合索引进行索引覆盖
3. ﻿﻿﻿如果索引失效，可以使用手动回表的技巧，强制走索引
4. ﻿﻿深分页的性能问题也可以通过业务方法解决，如限制查询页数 

### MySQL普通索引和唯一索引
你分得清MySQL普通索引和唯一索引了吗？
https://developer.aliyun.com/article/768845
由于唯一索引用不了change buffer的优化机制，因此如果业务可以接受，从性能角度，推荐优先考虑非唯一索引。



Comments for authors As you prepare your review, please focus on the following key areas and provide detailed comments that support your assessments:

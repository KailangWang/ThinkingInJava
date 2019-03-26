/**
 * controller
 *
 * @author Administrator
 */
@RestController
@RequestMapping("/goods")
public class GoodsController {

    @Reference
    private GoodsService goodsService;

//	@Reference
//	private ItemSearchService itemSearchService;更新索引库的操作全交给消息中间件了，不需要dubbo远程调用了

    @Reference
    private ItemPageService itemPageService;


    /**
     * 返回全部列表
     *
     * @return
     */
    @RequestMapping("/findAll")
    public List<TbGoods> findAll() {
        return goodsService.findAll();
    }


    /**
     * 返回全部列表
     *
     * @return
     */
    @RequestMapping("/findPage")
    public PageResult findPage(int page, int rows) {
        return goodsService.findPage(page, rows);
    }

    /**
     * 增加
     *
     * @param goods
     * @return
     */
    @RequestMapping("/add")
    public Result add(@RequestBody Goods goods) {
        try {
            goods.getGoods().setSellerId(SecurityContextHolder.getContext().getAuthentication().getName());
            goodsService.add(goods);
            return new Result(true, "增加成功");
        } catch (Exception e) {
            e.printStackTrace();
            return new Result(false, "增加失败");
        }
    }

    /**
     * 修改
     *
     * @param goods
     * @return
     */
    @RequestMapping("/update")
    public Result update(@RequestBody Goods goods) {
        try {
            goodsService.update(goods);
            return new Result(true, "修改成功");
        } catch (Exception e) {
            e.printStackTrace();
            return new Result(false, "修改失败");
        }
    }

    /**
     * 获取实体
     *
     * @param id
     * @return
     */
    @RequestMapping("/findOne")
    public Goods findOne(Long id) {
        return goodsService.findOne(id);
    }


    @Autowired
    @Qualifier(value = "solrDeleteQueueDestination")//注入点对点模式删除索引的目的地
    private Destination solrDeleteQueueDestination;

    /**
     * 批量删除
     *
     * @param ids
     * @return
     */
    @RequestMapping("/delete")
    public Result delete(Long[] ids) {
        try {
            goodsService.delete(ids);
            //删除商品SPU和对应的SKU
//			itemSearchService.deleteByGoodsId(Arrays.asList(ids));放弃使用dubbo远程删除索引
            String s = JSON.toJSONString(ids);
            jmsTemplate.convertAndSend(solrDeleteQueueDestination, s);
            return new Result(true, "删除成功");
        } catch (Exception e) {
            e.printStackTrace();
            return new Result(false, "删除失败");
        }
    }

    /**
     * 查询+分页
     *
     * @param
     * @param page
     * @param rows
     * @return
     */
    @RequestMapping("/search")
    public PageResult search(@RequestBody TbGoods goods, int page, int rows) {
        return goodsService.findPage(goods, page, rows);
    }


    @Autowired
    private JmsTemplate jmsTemplate;

    @Autowired
    @Qualifier(value = "solrQueueDestination")//注入点对点模式的目的地
    private Destination solrQueueDestination;

    @Autowired
    @Qualifier(value = "freeMarkerTopicDestination")//注入点对点模式的目的地
    private Destination freeMarkerTopicDestination;

    /***
     * 商品审核
     * @param ids 被审核的商品ID
     * @param status 审核的状态
     * @return
     */
    @RequestMapping("/updateStatus")
    public Result updateStatus(Long[] ids, String status) {
        try {
            goodsService.updateStatus(ids, status);
            //更新商品状态时，如果是审核通过，意味着有新商品上线了，可以被搜索到，需要更新到索引库
            if (status.equals("1")) {
                //查询SPU对应的SKU列表
                List<TbItem> itemList = goodsService.findItemListByGoodsId(ids);
                if (itemList.size() > 0) {
                    //有SKU数据,调用更新索引服务
//					itemSearchService.updateIndex(itemList);放弃dubbo远程调用服务，改用消息中间件
                    String jsonString = JSON.toJSONString(itemList);
                    jmsTemplate.convertAndSend(solrQueueDestination, jsonString);

                    //生成静态页面
//					for (Long id : ids) {
//					//根据SPU生成对应的静态页面
//						itemPageService.genHtml(id);
//					}
                    jmsTemplate.convertAndSend(freeMarkerTopicDestination, ids);//ids是Long[]，也是object类型


                } else {//没找到对应的SKU数据
                    System.out.println("没有对应SKU数据");
                }
            }
            return new Result(true, "成功");
        } catch (Exception e) {
            e.printStackTrace();
            return new Result(false, "失败");
        }
    }

}

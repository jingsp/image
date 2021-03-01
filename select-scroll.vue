export default {
  bind(el, binding) {   //  eslint-disable-line
    // 获取滚动页面DOM
    const SCROLL_DOM = el.querySelector('.el-select-dropdown .el-select-dropdown__wrap');
    let scrollPosition = 0;
    SCROLL_DOM.addEventListener('scroll', () => {
      // 得到滚动方向
      const diff = SCROLL_DOM.scrollTop - scrollPosition;
      scrollPosition = SCROLL_DOM.scrollTop;
      // 得到当前距离滚动区域底部的距离
      const bottom = SCROLL_DOM.scrollHeight - SCROLL_DOM.scrollTop - SCROLL_DOM.clientHeight;
      // if (bottom < 100) {
      //   binding.value(diff);
      // }
      // console.log(diff, 'diff', SCROLL_DOM.scrollTop, bottom);
      // 如果向下滚动 并且距离底部小于100px
      if (diff > 0 && bottom < 100) {
        binding.value({
          direction: 'bottom', diff, el: SCROLL_DOM, middlePosition: SCROLL_DOM.scrollHeight / 2,
        });
      } else if (diff < 0 && SCROLL_DOM.scrollTop < 100) {
        // console.log(22222);
        binding.value({
          direction: 'top', diff, el: SCROLL_DOM, middlePosition: SCROLL_DOM.scrollHeight / 2,
        });
      }
      // 如果向上滚动 并且距离顶部小于100px
    });
  },
};

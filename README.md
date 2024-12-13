# wordpress-action-timer
Measure wordpress' execution time for each action

paste into functions.php :


if(isset($_GET['slow'])){
    class WhatIsSoSlow {

        public $data = array();
    
        function __construct() {
            add_action( 'all', array( $this, 'filter_start' ) );
            add_action( 'shutdown', array( $this, 'results' ) );
        }
    
        // This runs first for all actions and filters.
        // It starts a timer for this hook.
        public function filter_start() {
            $current_filter = current_filter();
            
            $this->data[ $current_filter ][]['start'] = microtime( true );
    
            add_filter( $current_filter, array( $this, 'filter_end' ), 99999 );
        }
    
        // This runs last (hopefully) for each hook and records the end time.
        // This has problems if a hook fires inside of itself since it assumes
        // the last entry in the data key for this hook is the matching pair.
        public function filter_end( $filter_data = null ) {
            $current_filter = current_filter();
            
            remove_filter( $current_filter, array( $this, 'filter_end' ), 99999 );
    
            end( $this->data[ $current_filter ] );
    
            $last_key = key( $this->data[ $current_filter ] );
    
            $this->data[ $current_filter ][ $last_key ]['stop'] = microtime( true );
    
            return $filter_data;
        }
    
        // Processes the results and var_dump()'s them. TODO: Debug bar panel?
        public function results() {
            $results = array();
    
            foreach ( $this->data as $filter => $calls ) {
                foreach ( $calls as $call ) {
                    // Skip filters with no end point (i.e. the hook this function is hooked into)
                    if ( ! isset( $call['stop'] ) )
                        continue;
    
                    if ( ! isset( $results[ $filter ] ) )
                        $results[ $filter ] = 0;
    
                    $results[ $filter ] = $results[ $filter ] + ( $call['stop'] - $call['start'] );
                }
            }
    
            asort( $results, SORT_NUMERIC );
    
            $results = array_reverse( $results );
            echo "RRESULTSS";
           ?>
           <table>
            <?php 
            asort($results);
            foreach ($results as $k => $v) {
                ?>
                <tr>
<td><?php echo $k ?></td>
<td><?php echo $v ?></td>
                </tr>
                <?php
            } ?>
           </table>
           <?php
        }
    }
    
    new WhatIsSoSlow();
}

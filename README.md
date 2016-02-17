# Invoice-project
York University Object Oriented Project

note
	description: "Summary description for {MODEL}."
	author: ""
	date: "$Date$"
	revision: "$Revision$"

class
	MODEL

inherit
	ANY
	redefine out end

create {MODEL_ACCESS}
	make

feature {NONE} -- Initialization
	make
			-- Initialization for `Current'.
		do
			create s.make_from_string ("ok")
			create my_stock.make_empty
			create product.make_empty
			create invoice_l.make_empty
			i := 0
		end

feature -- model attributes
	s : STRING
	i : INTEGER
	my_stock: STOCK [STRING]
	product: PRODUCT [STRING]
	invoice_l: INVOICE_LIST [STRING]

feature -- model operations
	default_update
			-- Perform update to the model state.
		do
			i := i + 1
			create s.make_from_string ("default model state ")
			s.append ("(")
			s.append (i.out)
			s.append (")")
		end
	add_product(a_product: STRING ; quantity: INTEGER)
		require
			positive_quantity_only:
				quantity > 0
			product_type_exists:
				product.has_type (a_product)
    	do
			my_stock.extend (a_product,quantity)
		ensure
			a_product_exist_in_stock:
				my_stock.has (a_product)
			a_product_have_at_least_quantity_in_stock:
				my_stock.occurrences (a_product) >= quantity
    	end
    add_type(product_id: STRING)
    	require
    		product_id_must_non_empty_string:
    			not product_id.is_empty
    		product_type_dont_exists:
    			not(product.has_type (product_id))
    	do
			product.add_type (product_id)
		ensure
			id_added_to_product:
				product.has_type (product_id)
    	end
    add_order(a_order: ARRAY[TUPLE[pid: STRING; no: INTEGER]])
    	require
    		a_id_no_more_than_10000:
				not is_order_id_full
    		a_order_must_non_empty:
    			not a_order.is_empty
    		a_order_quantity_must_be_positive:
    			across a_order as ao all ao.item.no > 0 end
    		product_type_do_exists:
    			is_ordered_product_exist(a_order)
    		no_dupicate_in_a_order:
    			not is_there_duplicate_in(a_order)
    		a_order_is_subset_of_stork:
    			is_subset_of_stock(a_order)
    	do
			invoice_l.new_invoice (a_order)
			my_stock.remove_all (create {STOCK[STRING]}.make_from_tupled_array (a_order))
		ensure
			invoice_added_to_list:
				invoice_l.my_array[invoice_l.get_last_added_id].bag_equal (create {MY_INVOICE[STRING]}.make_from_tupled_array (a_order));
			stock_decrease:
				my_stock.is_subset_of (old my_stock)
    	end
    cancel_order(order_id: INTEGER)
    	require
    		order_id_do_exists:
    			order_id_exist(order_id)
    	do
			my_stock.add_all (create {STOCK[STRING]}.make_from_tupled_array (invoice_l.tuple_array (order_id)))
			invoice_l.cancel_invoice (order_id)
		ensure
			stock_increase:
				(old my_stock).is_subset_of (my_stock)
			invoice_cancelled_in_list:
				not invoice_l.is_id_exist (order_id)
    	end
    invoice(order_id: INTEGER)
    	require
    		order_id_do_exists:
    			order_id_exist(order_id)
    		order_id_not_already_invoiced:
    			not is_id_already_invoice(order_id)
    	do
			invoice_l.set_invoice (order_id)
		ensure
			invoice_status_is_invoiced:
				is_id_already_invoice(order_id)
    	end
    nothing
    	do

    	end
feature -- commands
	set_message(m: STRING)
		do
			s.copy (m)
		end
feature -- queries
	order_id_exist (a_id: INTEGER):BOOLEAN
		do
			Result := invoice_l.is_id_exist (a_id)
		end
	is_id_already_invoice(a_id: INTEGER):BOOLEAN
		do
			Result := invoice_l.is_id_already_invoice(a_id)
		end
	is_order_id_full: BOOLEAN
		do
			Result := invoice_l.is_invoice_sequence_over_10000
		end
	is_subset_of_stock(a_order: ARRAY[TUPLE[STRING,INTEGER]]): BOOLEAN
		do
			Result := create {STOCK[STRING]}.make_from_tupled_array (a_order) |<: my_stock
		end
	is_ordered_product_exist (a_order: ARRAY[TUPLE[pid: STRING; no: INTEGER]]): BOOLEAN
    	do
    		Result := across a_order as a all product.has_type (a.item.pid) end
    	end
	is_there_duplicate_in(a_order: ARRAY[TUPLE[pid: STRING; no: INTEGER]]): BOOLEAN
		local
			temp_array: ARRAY[STRING]
			index: INTEGER
		do
			Result := false
			index := 1
			create temp_array.make_empty
			temp_array.compare_objects
			across a_order as a
				loop
					if temp_array.has (a.item.pid) then
						Result := true
					end
					temp_array.force (a.item.pid, index)
					index := index +1
				end
		end
	has_type(a_product: STRING): BOOLEAN
		do
			Result := product.has_type (a_product)
		end
	stock_out: STRING
		do
			create Result.make_empty
			across my_stock.array_out as products loop
				Result.append (products.item.g + "->" + products.item.i.out + ",")
			end
			Result.remove_tail (1)
		end
	product_out: STRING
		do
			create Result.make_from_string (" ")
			across product.products as products loop
				Result.append (products.item + ",")
			end
			Result.remove_tail (1)
		end
	out : STRING
		do
			Result :="  report:      " + s +
				   "%N  id:          " + invoice_l.get_last_added_id.out +
				   "%N  products:   " + product_out +
				   "%N  stock:       " + stock_out +
				   "%N  orders:      " + invoice_l.orders_out +
				   "%N  carts:       " + invoice_l.out +
				   "%N  order_state: " + invoice_l.state_out
		end

	invariant
		all_type_exist_in_stock_must_exist_in_product:
			across my_stock as ms all product.has_type (ms.item) end
end
